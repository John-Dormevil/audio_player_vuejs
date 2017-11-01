<template>
	<div class="column">
		<div class="box">
			<div class="audio green-audio-player" @mousemove="mouveSliderH" @mouseleave="disableDraggable" ref="greenaudioplayer">
				<div class="loading" v-if="audio.isLoading">
					<div class="spinner"></div>
				</div>
				<div class="play-pause-btn" v-else @click="togglePlayPauseBtn">
					<svg xmlns="http://www.w3.org/2000/svg" width="18" height="24" viewBox="0 0 18 24">
						<path fill="#566574" fill-rule="evenodd" :d="audio.playPauseBtn" class="play-pause-icon" id="playPause" />
					</svg>
				</div>

				<div class="controls">
					<span class="current-time" ref="currenttime">{{ audio.formatedTime }}</span>
					<div class="slider" data-direction="horizontal" @click="findProperElmt" ref="slider">
						<div class="progress" ref="progress">
							<div class="pin" id="progress-pin" ref="progresspin"
							@mousedown="activeDraggable" @mouseup="disableDraggable"></div>
						</div>
					</div>
					<span class="total-time">{{ audio.totalTime }}</span>
				</div>

				<div class="volume">
					<div class="volume-btn">
						<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
							<path fill="#566574" fill-rule="evenodd" d="M14.667 0v2.747c3.853 1.146 6.666 4.72 6.666 8.946 0 4.227-2.813 7.787-6.666 8.934v2.76C20 22.173 24 17.4 24 11.693 24 5.987 20 1.213 14.667 0zM18 11.693c0-2.36-1.333-4.386-3.333-5.373v10.707c2-.947 3.333-2.987 3.333-5.334zm-18-4v8h5.333L12 22.36V1.027L5.333 7.693H0z" id="speaker" />
						</svg>
					</div>
					<div class="volume-controls hidden">
						<div class="slider" data-direction="vertical">
							<div class="progress">
								<div class="pin" id="volume-pin" data-method="changeVolume"></div>
							</div>
						</div>
					</div>
				</div>

				<audio crossorigin ref="player" @canplay="canPlayNow"
				@timeupdate="showCurrentTime" @ended="reset" @loadedmetadata="totalizer">
					<source src="https://s3-us-west-2.amazonaws.com/s.cdpn.io/355309/Swing_Jazz_Drum.mp3" type="audio/mpeg">
				</audio>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	data () {
		return {
			audio: {
				player: HTMLAudioElement,
				isLoading: true,
				isPlaying: false,
				playPauseBtn: 'M18 12L0 24V0',
				progress: HTMLDivElement,
				currentTime: HTMLSpanElement,
				progresspin: HTMLDivElement,
				formatedTime: '0:00',
				totalTime: '0:00',
				slider: HTMLDivElement,
				duration: 0,
				isDraggable: false,
				greenaudioplayer: HTMLDivElement
			}
		}
	},
	methods: {
		play () {
			this.audio.isPlaying = true
			this.audio.playPauseBtn = 'M0 0h6v24H0zM12 0h6v24h-6z'
			this.audio.player.play()
		},
		pause: function () {
			this.audio.isPlaying = false
			this.audio.playPauseBtn = 'M18 12L0 24V0'
			this.audio.player.pause()
		},
		togglePlayPauseBtn () {
			if (this.audio.isPlaying) {
				this.pause()
			} else {
				this.play()
			}
		},
		canPlayNow () {
			this.audio.isLoading = false
		},
		showCurrentTime () {
			var current = this.audio.player.currentTime
			var percent = (current / this.audio.player.duration) * 100
			var ft = this.formatTime(current)
			this.audio.progress.style.width = percent + '%'
			this.audio.progresspin.style.left = percent + '%'
			this.audio.formatedTime = ft
		},
		formatTime (time) {
			var min = Math.floor(time / 60)
			var sec = Math.floor(time % 60)
			return min + ':' + ((sec < 10) ? ('0' + sec) : sec)
		},
		totalizer () {
			var duration = this.audio.player.duration
			var total = this.formatTime(duration)
			this.audio.totalTime = total
		},
		initializer () {
			this.audio.progresspin.style.left = '0%'
		},
		findProperElmt (event) {
			var el = event.target
			var centerPin = 8
			if (el.className === this.audio.slider.className) {
				this.calculeNewTime(el, event, centerPin)
			}
			if (el.className === this.audio.progresspin.className) {
				centerPin = this.getOffset(el).with / 2
				el = event.path[2]
				this.calculeNewTime(el, event, centerPin)
			}
			if (el.className === this.audio.greenaudioplayer.className) {
				el = event.path[0].children[1].children[1]
				this.calculeNewTime(el, event, centerPin)
			}
		},
		calculeNewTime (el, event, centerPin) {
			var px = event.clientX
			var elCoo = this.getOffset(el)
			var progressClickPosition = (px - elCoo.left - centerPin) / (elCoo.right - elCoo.left)
			this.rewind(progressClickPosition)
			return progressClickPosition
		},
		rewind (progressClickPosition) {
			var total = this.audio.player.duration
			var calculePosition = total * progressClickPosition
			this.audio.player.currentTime = calculePosition
		},
		getOffset (el) {
			var x = 0
			var y = 0
			var r = 0
			var w = el.offsetWidth
			while (el && !isNaN(el.offsetLeft) && !isNaN(el.offsetTop)) {
				x += el.offsetLeft - el.scrollLeft
				y += el.offsetTop - el.scrollTop
				el = el.offsetParent
			}
			r = w + x
			return { top: y, left: x, right: r, with: w }
		},
		mouveSliderH (event) {
			// console.log(event.clientX)
			if (this.audio.isDraggable) {
				this.findProperElmt(event)
			}
		},
		activeDraggable () {
			this.audio.isDraggable = true
		},
		disableDraggable () {
			this.audio.isDraggable = false
		},
		reset () {
			this.audio.progresspin.style.left = '0%'
			this.audio.progress.style.width = '0%'
			this.audio.playPauseBtn = 'M18 12L0 24V0'
		}
	},
	mounted () {
		this.audio.player = this.$refs.player
		this.audio.progress = this.$refs.progress
		this.audio.currentTime = this.$refs.currenttime
		this.audio.progresspin = this.$refs.progresspin
		this.audio.slider = this.$refs.slider
		this.audio.greenaudioplayer = this.$refs.greenaudioplayer
		this.initializer()
	}
}
// var audioPlayer = document.querySelector('.green-audio-player')
// var playPause = audioPlayer.querySelector('#playPause')
// var playpauseBtn = audioPlayer.querySelector('.play-pause-btn')
// var loading = audioPlayer.querySelector('.loading')
// var progress = audioPlayer.querySelector('.progress')
// var sliders = audioPlayer.querySelectorAll('.slider')
// var volumeBtn = audioPlayer.querySelector('.volume-btn')
// var volumeControls = audioPlayer.querySelector('.volume-controls')
// var volumeProgress = volumeControls.querySelector('.slider .progress')
// var player = audioPlayer.querySelector('audio')
// var currentTime = audioPlayer.querySelector('.current-time')
// var totalTime = audioPlayer.querySelector('.total-time')
// var speaker = audioPlayer.querySelector('#speaker')

// var draggableClasses = ['pin']
// var currentlyDragged = null

// window.addEventListener('mousedown', function (event) {
// 	if (!isDraggable(event.target)) return false

// 	currentlyDragged = event.target
// 	let handleMethod = currentlyDragged.dataset.method

// 	this.addEventListener('mousemove', window[handleMethod], false)

// 	window.addEventListener('mouseup', () => {
// 		currentlyDragged = false
// 		window.removeEventListener('mousemove', window[handleMethod], false)
// 	}, false)
// })

// playpauseBtn.addEventListener('click', togglePlay)
// player.addEventListener('timeupdate', updateProgress)
// player.addEventListener('volumechange', updateVolume)
// player.addEventListener('loadedmetadata', () => {
// 	totalTime.textContent = formatTime(player.duration)
// })
// player.addEventListener('canplay', makePlay)
// player.addEventListener('ended', function () {
// 	playPause.attributes.d.value = 'M18 12L0 24V0'
// 	player.currentTime = 0
// })

// volumeBtn.addEventListener('click', () => {
// 	volumeBtn.classList.toggle('open')
// 	volumeControls.classList.toggle('hidden')
// })

// window.addEventListener('resize', directionAware)

// sliders.forEach(slider => {
// 	let pin = slider.querySelector('.pin')
// 	slider.addEventListener('click', window[pin.dataset.method])
// })

// directionAware()

// function isDraggable (el) {
// 	let canDrag = false
// 	let classes = Array.from(el.classList)
// 	draggableClasses.forEach(draggable => {
// 		if (classes.indexOf(draggable) !== -1) {
// 			canDrag = true
// 		}
// 	})
// 	return canDrag
// }

// function inRange (event) {
// 	let rangeBox = getRangeBox(event)
// 	let rect = rangeBox.getBoundingClientRect()
// 	let direction = rangeBox.dataset.direction
// 	var min, max
// 	if (direction === 'horizontal') {
// 		min = rangeBox.offsetLeft
// 		max = min + rangeBox.offsetWidth
// 		if (event.clientX < min || event.clientX > max) return false
// 	} else {
// 		min = rect.top
// 		max = min + rangeBox.offsetHeight
// 		if (event.clientY < min || event.clientY > max) return false
// 	}
// 	return true
// }

// function updateProgress () {
// 	var current = player.currentTime
// 	var percent = (current / player.duration) * 100
// 	progress.style.width = percent + '%'

// 	currentTime.textContent = formatTime(current)
// }

// function updateVolume () {
// 	volumeProgress.style.height = player.volume * 100 + '%'
// 	if (player.volume >= 0.5) {
// 		speaker.attributes.d.value = 'M14.667 0v2.747c3.853 1.146 6.666 4.72 6.666 8.946 0 4.227-2.813 7.787-6.666 8.934v2.76C20 22.173 24 17.4 24 11.693 24 5.987 20 1.213 14.667 0zM18 11.693c0-2.36-1.333-4.386-3.333-5.373v10.707c2-.947 3.333-2.987 3.333-5.334zm-18-4v8h5.333L12 22.36V1.027L5.333 7.693H0z'
// 	} else if (player.volume < 0.5 && player.volume > 0.05) {
// 		speaker.attributes.d.value = 'M0 7.667v8h5.333L12 22.333V1L5.333 7.667M17.333 11.373C17.333 9.013 16 6.987 14 6v10.707c2-.947 3.333-2.987 3.333-5.334z'
// 	} else if (player.volume <= 0.05) {
// 		speaker.attributes.d.value = 'M0 7.667v8h5.333L12 22.333V1L5.333 7.667'
// 	}
// }

// function getRangeBox (event) {
// 	let rangeBox = event.target
// 	let el = currentlyDragged
// 	if (event.type === 'click' && isDraggable(event.target)) {
// 		rangeBox = event.target.parentElement.parentElement
// 	}
// 	if (event.type === 'mousemove') {
// 		rangeBox = el.parentElement.parentElement
// 	}
// 	return rangeBox
// }

// function getCoefficient (event) {
// 	let slider = getRangeBox(event)
// 	let rect = slider.getBoundingClientRect()
// 	let K = 0
// 	if (slider.dataset.direction === 'horizontal') {
// 		let offsetX = event.clientX - slider.offsetLeft
// 		let width = slider.clientWidth
// 		K = offsetX / width
// 	} else if (slider.dataset.direction === 'vertical') {
// 		let height = slider.clientHeight
// 		var offsetY = event.clientY - rect.top
// 		K = 1 - offsetY / height
// 	}
// 	return K
// }

// function setDuration (event) {
// 	if (inRange(event)) {
// 		player.currentTime = player.duration * getCoefficient(event)
// 	}
// }
// setDuration(event)

// function changeVolume (event) {
// 	if (inRange(event)) {
// 		player.volume = getCoefficient(event)
// 	}
// }
// changeVolume(event)

// function formatTime (time) {
// 	var min = Math.floor(time / 60)
// 	var sec = Math.floor(time % 60)
// 	return min + ':' + ((sec < 10) ? ('0' + sec) : sec)
// }

// function togglePlay () {
// 	if (player.paused) {
// 		playPause.attributes.d.value = 'M0 0h6v24H0zM12 0h6v24h-6z'
// 		player.play()
// 	} else {
// 		playPause.attributes.d.value = 'M18 12L0 24V0'
// 		player.pause()
// 	}
// }

// function makePlay () {
// 	playpauseBtn.style.display = 'block'
// 	loading.style.display = 'none'
// }

// function directionAware () {
// 	if (window.innerHeight < 250) {
// 		volumeControls.style.bottom = '-54px'
// 		volumeControls.style.left = '54px'
// 	} else if (audioPlayer.offsetTop < 154) {
// 		volumeControls.style.bottom = '-164px'
// 		volumeControls.style.left = '-3px'
// 	} else {
// 		volumeControls.style.bottom = '52px'
// 		volumeControls.style.left = '-3px'
// 	}
// }
</script>

<style>
.audio.green-audio-player {
	width: 400px;
	min-width: 300px;
	height: 56px;
	box-shadow: 0 4px 16px 0 rgba(0, 0, 0, 0.07);
	display: flex;
	justify-content: space-between;
	align-items: center;
	padding-left: 24px;
	padding-right: 24px;
	border-radius: 4px;
	user-select: none;
	-webkit-user-select: none;
	background-color: #fff;
}

.play-pause-btn {
	cursor: pointer;
}

.spinner {
	width: 18px;
	height: 18px;
	background-image: url(https://s3-us-west-2.amazonaws.com/s.cdpn.io/355309/loading.png);
	background-size: cover;
	background-repeat: no-repeat;
	animation: spin 0.4s linear infinite;
}

.slider {
	flex-grow: 1;
	background-color: #d8d8d8;
	cursor: pointer;
	position: relative;
}

.progress {
	background-color: #44bfa3;
	border-radius: inherit;
	pointer-events: none;
}

.pin {
	height: 16px;
	width: 16px;
	border-radius: 8px;
	background-color: #44bfa3;
	position: absolute;
	pointer-events: all;
	box-shadow: 0px 1px 1px 0px rgba(0, 0, 0, 0.32);
}

.controls {
	font-family: "Roboto", sans-serif;
	font-size: 16px;
	line-height: 18px;
	color: #55606e;
	display: flex;
	flex-grow: 1;
	justify-content: space-between;
	align-items: center;
	margin-left: 24px;
	margin-right: 24px;
}

.controls .slider {
	margin-left: 16px;
	margin-right: 16px;
	border-radius: 2px;
	height: 4px;
}

.controls .slider .progress {
	width: 0;
	height: 100%;
}

.controls .slider .progress .pin {
	right: -8px;
	top: -6px;
}

.controls span {
	cursor: default;
}

.volume {
	position: relative;
}

.volume .volume-btn {
	cursor: pointer;
}

.volume .volume-btn.open path {
	fill: #44bfa3;
}

.volume .volume-controls {
	width: 30px;
	height: 135px;
	background-color: rgba(0, 0, 0, 0.62);
	border-radius: 7px;
	position: absolute;
	left: -3px;
	bottom: 52px;
	flex-direction: column;
	align-items: center;
	display: flex;
}

.volume .volume-controls.hidden {
	display: none;
}

.volume .volume-controls .slider {
	margin-top: 12px;
	margin-bottom: 12px;
	width: 6px;
	border-radius: 3px;
}

.volume .volume-controls .slider .progress {
	bottom: 0;
	height: 100%;
	width: 6px;
}

.volume .volume-controls .slider .progress .pin {
	left: -5px;
	top: -8px;
}

svg,
img {
	display: block;
}

.box {
	display: flex;
	justify-content: center;
	align-items: center;
	background: #f8ffae;
	background: -webkit-linear-gradient(-65deg, #43c6ac, #f8ffae);
	background: linear-gradient(-65deg, #43c6ac, #f8ffae);
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

@keyframes spin {
	from {
		transform: rotateZ(0);
	}
	to {
		transform: rotateZ(1turn);
	}
}
</style>


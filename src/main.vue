<template lang="pug">
	.player.noselect(
		@mousemove.self='moveCursorMarker'
		ref='player'
	)
		audio(
			v-bind:src='src' ref='audio'
			@canplaythrough='trackerClickable = true'
		)
		.tracker(ref='tracker')
		.play-button(
			@click='togglePlaying'
			:class='{ pause : audioPlaying }'
		)
		.timer {{ currentTime }}/{{ GetTime( duration ) }}
		.cursor-marker(
			ref='marker'
			@click='updateTime'
		)
			span {{ cursorTime.string }}
</template>

<script>
export default {
	props : {
		src : {
			required : true
		},
		duration : {
			default : null
		}
	},

	data : () => ( {
		audioPlaying     : false,
		trackerClickable : false,
		trackerAnimation : null,
		currentTime      : 0,
		cursorTime       : {
			string : null,
			ms     : null,
		}
	} ),

	methods : {

		togglePlaying() {

			if ( this.audioPlaying ) {
				this.audioPlaying = false;

				this.$refs.audio.pause();
			}

			else {
				this.audioPlaying = true;

				this.$refs.audio.play();

				this.trackerAnimation = window.requestAnimationFrame( this.updateTracker );
			}

		},

		updateTracker() {

			if ( !this.$refs.audio ) {
				return;
			}

			const { currentTime } = this.$refs.audio;
			let { duration }      = this.$refs.audio;

			if ( isNaN( duration ) || duration === Infinity ) {
				duration = ( this.duration / 1000 );
			}

			const percent = ( currentTime / duration ) * 100;

			this.currentTime               = GetTime( currentTime * 1000 );
			this.$refs.tracker.style.width = `${percent}%`;

			if ( percent !== 100 ) {
				this.trackerAnimation = window.requestAnimationFrame( this.updateTracker );

				return;
			}

			this.trackerAnimation = null;
			this.audioPlaying     = false;

		},

		moveCursorMarker( e ) {

			const w = this.$refs.player.getBoundingClientRect().width;
			const l = e.offsetX;

			const percent = ( l / w );

			const duration = ( () => {
				const d = this.$refs.audio.duration;

				if ( isNaN( d ) || d === Infinity ) {
					return this.duration;
				}

				return d * 1000;

			} )();

			const time       = ( duration * percent );
			const cursorTime = ( ( str ) => {

				if ( time < ( 60 * 1000 ) ) {
					const ms = Math.floor( time % 1000 );

					return `${str}.${ms}`;
				}

				return str;

			} )( GetTime( time ) );

			this.cursorTime = {
				string : cursorTime,
				ms     : time
			};

			this.$refs.marker.style.left = `${l}px`;

		},

		updateTime() {

			this.$refs.audio.currentTime = ( this.cursorTime.ms / 1000 );

			if ( this.trackerAnimation === null ) {
				this.updateTracker();
			}

		},

		GetTime,

	},

	beforeDestroy() {
		if ( this.trackerAnimation ) {
			window.cancelAnimationFrame( this.trackerAnimation );
		}
	}
}
</script>

<style lang="scss">
	 $opaqueLightBlue : rgba(46, 84, 145, 0.15);
	 $lightBlue       : #2E5491;


	.player {
		position: relative;
		height: 70px;
		z-index: 2;
		transition: opacity 0.2s ease;
		padding: 0 30px;
		display: flex;
		align-items: center;
		justify-content: space-between;
		background: $opaqueLightBlue;

		&.no-bg {
			background-color: none;
		}

		&:hover {

			.cursor-marker{

				&::before {
					height: 100%;
					width: 100%;
				}

				span {
					opacity: 0.5;
					top: 6px;
				}
			}
		}

		// hide cursor if we hover over another element.
		// for this to work .cursor-marker must be the
		// last child of .player
		> *:not(.cursor-marker):hover ~ .cursor-marker {

			&::before {
				opacity: 0;
			}

			span {
				opacity: 0;
			}
		}

		.cursor-marker {
			position: absolute;
			top: 0;
			bottom: 0;
			left: 0;
			transform: translate(-50%,-50%);
			width: 2px;

			&::before {
				content : ' ';
				opacity: 0.5;
				position: absolute;
				top: 50%;
				left: 50%;
				height: 0;
				width: 2;
				background-color: $lightBlue;
				transition: height 0.2s ease;
				will-change: height;
				transform: translateX(-50%);
			}

			span {
				position: absolute;
				top: 10px;
				left: 50%;
				transform: translateX(-50%);
				padding: 5px;
				border-radius: 3px;
				background-color: $lightBlue;
				color: white;
				opacity: 0;
				transition: opacity 0.2s ease, top 0.2s ease;
			}
		}

		.tracker {
			position: absolute;
			top: 0;
			bottom: 0;
			left: 0;
			background-color: $lightBlue;
			opacity: 0.25;
			z-index: 1;
			pointer-events: none;
		}

		.play-button {
			height: 45px;
			width: 45px;
			border-radius: 50%;
			background-color: $lightBlue;
			position: relative;
			cursor: pointer;
			z-index: 2;

			&:active {
				opacity: 0.8;
			}

			&.pause {

				&::before {
					transform: translate(calc(-50% - 5px), -50%);
					border-top-width: 0;
					border-bottom-width: 0;
					border-right-width: 0;
					height: 15px;
				}

				&::after {
					transform: translate(-50%, -50%);
					position: absolute;
					border-top-width: 0;
					border-bottom-width: 0;
					border-right-width: 0;
					height: 15px;
				}
			}

			&::before,
			&::after {
				transition: transform 0.2s ease, height 0.2s ease, border 0.2s ease;
				will-change: transform, height, border;
			}

			&::before {
				content: ' ';
				position: absolute;
				top: 50%;
				left: 51%;
				transform: translate(-50%, -50%);
				border: 5px solid white;
				border-top: 4px solid transparent;
				border-bottom: 4px solid transparent;
				border-right-width: 0;
				height: 8px;
			}

			&::after {
				content: ' ';
				position: absolute;
				top: 50%;
				left: 62%;
				transform: translate(-50%, -50%);
				border: 5px solid white;
				border-top: 4px solid transparent;
				border-bottom: 4px solid transparent;
				border-right: 0;
				opacity: 1;
				height: 0;
			}
		}

		.timer {
			padding: 5px;
			border-radius: 3px;
			background-color: $lightBlue;
			color: white;
			font-size: 12px;
			letter-spacing: 2px;
			z-index: 2;
		}
	}
</style>


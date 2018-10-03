audio-player

========

audio player vue component

## Installation

  `npm install @spacebartech/audio-player`

## Usage

  `import AudioPlayer from '@spacebartech/audio-player'`

  ```
  props : {
		src : {
      type     : String,
			required : true
		},
		duration : {
      type    : String,
			default : null
		}
	}

  audio-player(
    :src='audioSrc'
    :duration='duration'
  )

  ```

## Tests

  `npm test`

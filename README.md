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
      required : true
    },
    duration : {
      default : null
    }
  }

  audio-player(
    :src='audioSrc'
    :duration='duration'
  )

  ```

  ```
  Sass variables

  $primaryColor : #2E5491 !default;

  ```

## Tests

  `npm test`

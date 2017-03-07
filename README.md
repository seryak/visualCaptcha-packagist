[![Build Status](https://travis-ci.org/emotionLoop/visualCaptcha-packagist.svg?flat=true&branch=master)](https://travis-ci.org/emotionLoop/visualCaptcha-packagist)
[![Codacy](https://www.codacy.com/project/badge/8cc65dd81c9040a2a9ed262b07d4fa50)](https://www.codacy.com/app/bruno-bernardino/visualCaptcha-packagist)
[![Code Climate](https://codeclimate.com/github/emotionLoop/visualCaptcha-packagist/badges/gpa.svg)](https://codeclimate.com/github/emotionLoop/visualCaptcha-packagist)

# visualCaptcha-packagist

Packagist composer package for visualCaptcha's backend service


## Installation with Composer

You need PHP installed with [composer](https://getcomposer.org).
```
composer install emotionloop/visualcaptcha
```


## Run tests

You need [PHPUnit](https://phpunit.de) installed and then you can run
```
composer install && phpunit tests
```


## Usage

### Initialization

You have to initialize a session for visualCaptcha to inject the data it needs. You'll need this variable to start and verify visualCaptcha as well.

```
session_start();// Only needed once, at the beginning of your PHP files.
$session = new \visualCaptcha\Session( $namespace );
```
Where:

- `$namespace` is optional. It's a string and defaults to 'visualcaptcha'. You'll need to specifically set this if you're using more than one visualCaptcha instance in the same page, so the code can identify from which one is the validation coming from.

### Using Cache

You can use a backend Zend-Cache library options to store images on cache and avoid I/O.
You'll have to pass options parameter on the cosntructor as document in https://docs.zendframework.com/zend-cache/storage/adapter/.
By dafault it is disabled and you´ll have to pass true on the constructor on the 5th parameter.

```
$options = array(
                'adapter' => array(
                    'name'    => 'memory',
                    'options' => array('ttl' => 3600,
                        'namespace' => "captcha-service"),
                ),
                'plugins' => array( 
                    'exception_handler' => array('throw_exceptions' => false),
                ),
            );
        
$captchaWithCache = new \visualCaptcha\Captcha( $this->session,null,null,null, true,$options);
```

### Setting Routes for the front-end

You also need to set routes for `/start/:howmany`, `/image/:index`, and `/audio/:index`. These will usually look like:

```
// Populates captcha data into session object
// -----------------------------------------------------------------------------
// @param howmany is required, the number of images to generate
$app->get( '/start/:howmany', function( $howMany ) use( $app ) {
    $captcha = new \visualCaptcha\Captcha( $app->session );
    $captcha->generate( $howMany );
    $app->response[ 'Content-Type' ] = 'application/json';
    echo json_encode( $captcha->getFrontEndData() );
} );

// Streams captcha images from disk
// -----------------------------------------------------------------------------
// @param index is required, the index of the image you wish to get
$app->get( '/image/:index', function( $index ) use( $app ) {
    $captcha = new \visualCaptcha\Captcha( $app->session );
    if ( ! $captcha->streamImage(
            $app->response,
            $index,
            $app->request->params( 'retina' )
    ) ) {
        $app->pass();
    }
} );

// Streams captcha audio from disk
// -----------------------------------------------------------------------------
// @param type is optional and defaults to 'mp3', but can also be 'ogg'
$app->get( '/audio(/:type)', function( $type = 'mp3' ) use( $app ) {
    $captcha = new \visualCaptcha\Captcha( $app->session );
    if ( ! $captcha->streamAudio( $app->response, $type ) ) {
        $app->pass();
    }
} );
```

### Validating the image/audio

Here's how it'll usually look:

```
$session = new \visualCaptcha\Session();
$captcha = new \visualCaptcha\Captcha( $session );
$frontendData = $captcha->getFrontendData();

// If an image field name was submitted, try to validate it
if ( $imageAnswer = $app->request->params( $frontendData[ 'imageFieldName' ] ) ) {
  if ( $captcha->validateImage( $imageAnswer ) ) {
    // Image was valid.
  } else {
    // Image was submitted, but wrong.
  }
// Otherwise an audio field was submitted, so try to validate it
} else if ( $audioAnswer = $app->request->params( $frontendData[ 'audioFieldName' ] ) ) {
  if ( $captcha->validateAudio( $audioAnswer ) ) {
    // Audio answer was valid.
  } else {
    // Audio was submitted, but wrong.
  }
} else {
  // Apparently no fields were submitted, so the captcha wasn't filled.
}
```

### visualCaptcha/Session properties

- `$namespace`, String â This is private and will hold the namespace for each visualCaptcha instance. Defaults to 'visualcaptcha'.

### visualCaptcha/Session methods

- `__construct( $namespace )` â Initialize the visualCaptcha session.
- `clear()` â Will clear the session for the current namespace.
- `get( $key )` â Will return a value for the session's `$key`.
- `set( $key, $value )` â Set the `$value` for the session's `$key`.


### visualCaptcha/Captcha properties

- `$session`, Object that will have a reference for the session object.
  It will have .visualCaptcha.images, .visualCaptcha.audios, .visualCaptcha.validImageOption, and .visualCaptcha.validAudioOption.
- `$assetsPath`, Assets path. By default, it will be './assets'
- `$imageOptions`, All the image options.
  These can be easily overwritten or extended using addImageOptions( <Array> ), or replaceImageOptions( <Array> ). By default, they're populated using the ./images.json file
- `$audioOptions`, All the audio options.
  These can be easily overwritten or extended using addAudioOptions( <Array> ), or replaceAudioOptions( <Array> ). By default, they're populated using the ./audios.json file

### visualCaptcha/Captcha methods

You'll find more documentation on the code itself, but here's the simple list for reference.

- `__construct( $session, $assetsPath = null, $defaultImages = null, $defaultAudios = null )` â Initialize the visualCaptcha object.
- `generate( $numberOfOptions = 5 )` â Will generate a new valid option, within a `$numberOfOptions`.
- `streamAudio( $headers, $fileType )` â Stream audio file.
- `streamImage( $headers, $index, $isRetina )` â Stream image file given an index in the session visualCaptcha images array.
- `getFrontendData()` â Get data to be used by the frontend.
- `getValidImageOption()` â Get the current validImageOption.
- `getValidAudioOption()` â Get the current validAudioOption.
- `validateImage( $sentOption )` â Validate the sent image value with the validImageOption.
- `validateAudio( $sentOption )` â Validate the sent audio value with the validAudioOption.
- `getImageOptions()` â Return generated image options.
- `getImageOptionAtIndex( $index )` â Return generated image option at index.
- `getAudioOption()` â Alias for getValidAudioOption.
- `getAllImageOptions()` â Return all the image options.
- `getAllAudioOptions()` â Return all the audio options.


## License

View the [LICENSE](LICENSE) file.
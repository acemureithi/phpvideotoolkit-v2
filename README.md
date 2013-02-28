#PHPVideoToolkit V2...

...is a set of PHP classes aimed to provide a modular, object oriented and accessible interface for interacting with videos and audio through FFmpeg.

It also currently provides FFmpeg-PHP emulation in pure PHP so you wouldn't need to compile and install the module. As FFmpeg-PHP has not been updated since 2007 using FFmpeg-PHP with a new version of FFmpeg can often break the module. Using PHPVideoToolkits' emulation of FFmpeg-PHP's functionality allows you to upgrade FFmpeg without worrying about breaking existing funcitonality.

##Documentation

Extensive documentation and examples are bundled with the download and is available in the documentation directory.

##Usage

Whilst the extensive documentation covers just about everything, here are a few examples of what you can do.

###Configuring PHPVideoToolkit

PHPVideoToolkit requires some basic configuration and is one through the Factory class.

```php
namespace PHPVideoToolkit;

Factory::setDefaultVars('./tmp', '/opt/local/bin', 'ffmpeg', 'ffprobe');
```
###Accessing Data About FFmpeg

Simple demonstration about how to access information about FfmpegParser object.

```php
namespace PHPVideoToolkit;

$ffmpeg = Factory::ffmpegParser();
$is_available = $ffmpeg->isAvailable(); // returns boolean
$ffmpeg_version = $ffmpeg->getVersion(); // outputs something like - array('version'=>1.0, 'build'=>null)
	
```
###Accessing Data About media files

Simple demonstration about how to access information about media files using the MediaParser object.

```php
namespace PHPVideoToolkit;

$parser = Factory::mediaParser();
$data = $parser->getFileInformation('BigBuckBunny_320x180.mp4');
echo '<pre>'.print_r($data, true).'</pre>';
	
```
###Extract a Single Frame of a Video

The code below extracts a frame from the video at the 40 second mark.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->extractFrame(new Timecode(40))
	   			->save('./output/big_buck_bunny_frame.jpg');
```
###Extract Multiple Frames from a Segment of a Video

The code below extracts frames at the parent videos' frame rate from between 40 and 50 seconds. If the parent video has a frame rate of 24 fps then 240 images would be extracted from this code.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->extractFrames(new Timecode(40), new Timecode(50))
	   			->save('./output/big_buck_bunny_frame_%timecode.jpg');
```

###Extract Multiple Frames of a Video at 1 frame per second

There are two ways you can export at a differing frame rate from that of the parent video. The first is to use an output format to set the video frame rate.

```php
namespace PHPVideoToolkit;

$output_format = \PHPVideoToolkit\Factory::videoFormat('output');
$output_format->setFrameRate(1);

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->extractFrames(null, new Timecode(50)) // if null then the extracted segment starts from the begining of the video
	   			->save('./output/big_buck_bunny_frame_%timecode.jpg', $output_format);
```

The second is to use the $force_frame_rate option of the extractFrames function.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->extractFrames(new Timecode(50), null, 1) // if null then the extracted segment goes from the start timecode to the end of the video
	   			->save('./output/big_buck_bunny_frame_%timecode.jpg');
```

###Extracting Audio or Video Channels from a Video

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->extractAudio()->save('./output/big_buck_bunny.mp3');
// $output = $video->extractVideo()->save('./output/big_buck_bunny.mp4');
	   			
```

###Extracting a Segment of an Audio or Video file

The code below extracts a portion of the video at the from 2 minutes 22 seconds to 3 minutes. *Note the different settings for constructing a timecode.*

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->extractSegment(new Timecode('00:02:22.0', Timecode::INPUT_FORMAT_TIMECODE), new Timecode(180))
	   			->save('./output/big_buck_bunny.mp4');
```
###Spliting a Audio or Video file into multiple parts

There are multiple ways you can configure the split parameters. If an array is supplied as the first argument. It must be an array of either, all Timecode instances detailing the timecodes at which you wish to split the media, or all integers. If integers are supplied the integers are treated as frame numbers you wish to split at. You can however also split at even intervals by suppling a single integer as the first paramenter. That integer is treated as the number of seconds that you wish to split at. If you have a video that is 3 minutes 30 seconds long and set the split to 60 seconds, you will get 4 videos. The first three will be 60 seconds in length and the last would be 30 seconds in length.

The code below splits a video into multiple of equal length of 45 seconds each. 

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->split(45)
	   			->save('./output/big_buck_bunny_%timecode.mp4');
```
###Purging and then adding Meta Data

Unfortunately there is no way using FFmpeg to add meta data without re-encoding the file. There are other tools that can do that.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->purgeMetaData()
				->setMetaData('title', 'Hello World')
	   			->save('./output/big_buck_bunny.mp4');
```
###Non-Blocking Saves

The default/main save() function blocks PHP untill the encoding process has completed. This means that depending on the size of the media you are encoding it could leave your script running for a long time. To combat this you can call saveNonBlocking() to start the encoding process without blocking PHP.

However there are some caveats you need to be aware of before doing so. Once the non blocking process as started, if your PHP script closes PHPVideoToolkit can not longer "tidy up" temporary files or perform dynamic renaming of %index or %timecode output files. All repsonsibility is handed over to you. Of course, if you leave the PHP script open untill the encode has finished PHPVideoToolkit will do everything for you.

The code below is an example of how to manage a non-blocking save.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$process = $video->saveNonBlocking('./output/big_buck_bunny.mov');

// do something else important, db queries etc

while($process->isCompleted() === false)
{
	// do something more stuff in a loop.
	// doesn't have to be a loop, just an example.
	
	sleep(0.5);
}

if($process->hasError() === true)
{
	// an error was encountered, do something with it.
}
else
{
	// encoding has completed and no error was detected so 
	// we can get the output from the process.
	$output = $process->getOutput();
}

```

###Encoding with Progress Handlers

Whilst the code above from Non-Blocking Saves looks like it is a progress handler (and it is in a sense, but it doesn't provide data on the encode), progress handlers provide much more detailed information about the current encoding process.

PHPVideoToolkit allows you to monitor the encoding process of FFmpeg. This is done by using ProgressHandler objects. There are two types of progress handlers. ProgressHandlerNative and ProgressHandlerOutput. If your copy of FFmpeg is recent you will be able to use ProgressHandlerNative which uses FFmpegs '-progress' command to provide data, where was ProgressHandlerOutput relies on only the output of the command line buffer. Apart from that difference both handlers return the same data and act in the same way.

Progress Handlers can be made to block PHP or can be used in a non blocking fashion. They can even be utilized to work from a seperate script once the encoding has been initialised. However for purposes of this example the progress handlers are in the same script essentially blocking the PHP process. Again however, the two examples shown function very differently.

**Example 1. Callback in the handler constructor**

This example supplies the progress callback handler as a paramater to the constructor. This function is then called (every second, by default). Creating the callback in this way will block PHP and cannot be assigned as a progress handler when calling saveNonBlocking().

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');

$progress_handler = new ProgressHandlerNative(function($data)
{
	echo '<pre>'.print_r($data, true).'</pre>';
});

$output = $video->purgeMetaData()
				->setMetaData('title', 'Hello World')
	   			->save('./output/big_buck_bunny.mp4', null, Video::OVERWRITE_EXISTING, $progress_handler);
```

**Example 2. Probing the handler**

This example initialises a handler but does not supply a callback function. Instead you create your own method for creating a "progress loop" (or similar) and instead just call probe() on the handler.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');

$progress_handler = new ProgressHandlerNative();

$output = $video->purgeMetaData()
				->setMetaData('title', 'Hello World')
	   			->saveNonBlocking('./output/big_buck_bunny.mp4', null, Video::OVERWRITE_EXISTING, $progress_handler);
				
while($progress_handler->completed !== true)
{
	// note setting true in probe() automatically tells the probe to wait after the data is returned.
 	echo '<pre>'.print_r($progress_handler->probe(true), true).'</pre>';
}
				
```

So you see whilst the two examples look very similar and both block PHP, the second example does not need to block at all.


###Accessing Executed Commands and the Command Line Buffer

There may be instances where things go wrong and PHPVideoToolkit hasn't correctly prevented or reported any encoding/decoding errors, or, you may just want to log what is going on. You can access any executed commands and the command lines output fairly simply as the example below shows.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$output = $video->save('./output/big_buck_bunny.mov');
$process = $video->getProcess();

echo 'Expected Executed Command<br />';
echo '<pre>'.$process->getExecutedCommand().'</pre>';

echo 'Expected Command Line Buffer<br />';
echo '<pre>'.$process->getBuffer().'</pre>';
```

It's important to note, the the ExecBuffer object actually manipulates the raw command string given to it by the FfmpegProcess object. This is done so that the ExecBuffer can successfully track errors and process completion. The data returned by getExecutedCommand() and getBuffer() are values that are expected but not actual.

To get the actual executed command and buffer you can use the following.

```php
echo 'Actual Executed Command<br />';
echo '<pre>'.$process->getExecutedCommand(true).'</pre>';

echo 'Actual Command Line Buffer<br />';
echo '<pre>'.$process->getRawBuffer().'</pre>';
```

###Supplying custom commands

Because FFmpeg has a specific order in which certain commands need to be added there are a few functions you should be aware of. First of the code below shows you how to access the code FfmpegProcess object. The process object is itself a wrapper around the ProcessBuilder (helps to build queries) and ExceBuffer (executes and controls the query) objects.

The process object is passed by reference so any changes to the object are also made within the Video object.

```php
namespace PHPVideoToolkit;

$video  = Factory::video('BigBuckBunny_320x180.mp4');
$process = $video->getProcess();
				
```

Now you have access to the process object you can add specific commands to it. 

```php
// ... continued from above

$process->addPreInputCommand('-custom-command');
$process->addCommand('-custom-command-with-arg', 'arg value');
$process->addPostOutputCommand('-output-command', 'another value');
				
```

Now all of the example commands above will cause FFmpeg to fail, and they are jsut to illustrate a point.

- The function addPreInputCommand() adds commands to be given before the input command (-i) is added to the command string.
- The function addCommand() adds commands to be given after the input command (-i) is added to the command string.
- The function addPostOutputCommand() adds commands to be given after the output file is added to the command string.

To help explain it further, here is a simplified command string using the above custom commands.

```
/opt/bin/local/ffmpeg -custom-command -i '/your/input/file.mp4' -custom-command-with-arg 'arg value' '/your/output/file.mp4' -output-command 'another value'
```








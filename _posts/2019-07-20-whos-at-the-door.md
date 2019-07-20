---
layout: post
title: Who's at the door?
summary: Combining motion detection, Home Assistant, Node-RED and Amazon Rekognition
date: 2019-07-18 15:58
categories: blog
tags: ruby home_automation node-red aws
---

Home Automation is the Deadwood of the technological Wild West.  An illegal settlement, in violation of various treaties, run by murderers and thieves.  A modern Wild Bill Hickok killed when his Works With Nest integration is [suddenly sunset][1] by a cowardly Google.

One day Home Automation will be tamed.  Things will *Just Work*, you'll be able to buy automated lights, fans, thermostats, locks, and cameras that don't all have their own wireless standard, hub and app.  Or more than likely when a house is built all of this stuff will just be put in along with the water heater and electrical.  For now - its all kind of a hell ride.

For smart home hobbyists[^1], [Home Assistant][2] and [Node-RED][3] really help tie the room together.  The purpose of this post is to go into how I send a short video clip to my wife and I's phones whenever a person is detected at our front door.

The automation started modestly.  I thought I wanted to get a recording on my phone whenever our front door camera detected motion.  That evolved into only getting a clip when both my wife and I were away, so we weren't getting clips of each other walking out to check the mail.  But we *did* want to get a clip when we're home if motion was detected at night.  And finally I don't want a clip at all unless there's a person there - mostly due to getting 20 recordings a night of a mayfly buzzing around.

![Tomato time](/assets/2019_ha/tomato.png)

## The hardware

We have several [Axis cameras][13], which have a ton of options on board for motion detection, clip recording, etc.  The model I'm using for this is an M3026.  I imagine that future versions will have some pre-trained machine learning onboard so they can perform their own [hot dog/not hot dog][4] type detection.  

Really the only other hardware that matters is the Raspberry Pi3 that runs Home Assistant and Node-RED.  The camera is just connected to the local network via a [PoE switch][14].

## The automation

Here is the Node-RED flow for the automation. (source [here][16])

![node red flow](/assets/2019_ha/node-red.png)

1. Motion is detected by the camera, which is picked up by Home Assistant thus triggering the flow on Node-RED.  Whenever the camera detects motion it records a clip to /tmp/frontdoor.mkv on the Pi
1. The automation passes through a few logic gates before continuing.  If its night time OR Leah and I are both [away][5], the automation continues
1. ffmpeg converts the mkv from the camera into a mp4.  
1. The clip is uploaded to Amazon Rekognition using a ruby script.  If a person/persons are detected in the clip the automation continues
1. Finally the clip is sent to Leah and I via a [Telegram bot][6]

Theres also some error/debug logging.  I can run `sudo journalctl -f -u nodered -o cat` on the Pi to check the output of ffmpeg and the ruby script.

## Motion detection and clip recording

There are a few ways the clip can be recorded.  I initially had a [stream][15] setup for the camera in Home Assistant but this proved to be error prone.  Also when dealing with motion detection you really need to start the clip recording several seconds *before* motion is detected.  Otherwise you end up with a video of someone walking away.  Luckily Axis (and most IP cameras) have a "look behind" type feature for motion triggered recordings.

You're probably wondering why I have ffmpeg running to convert the clip.  That was the most frustrating part of this process.  There are a myriad of different ways you can send notifications from Home Assistant to a smartphone.  If you'd like to send a notification with an auto playing video clip - Telegram is your best option (at least in my experience[^2]).  In order to send a video clip through Telegram it *must* be an mp4 (which it then converts to a gif for... reasons).  And even though the Axis camera has the ability to produce mp4 files instead of mkv, I could not get these to send via Telegram.  

(╯°□°)╯︵ ┻━┻

That's where ffmpeg comes in.  Its entire purpose is to convert the video.  `ffmpeg -hide_banner -loglevel info -i /home/pi/tmp/frontdoor.mkv -codec copy /tmp/frontdoor.mp4 -y`

## Person detection

Finally, the last part of the automation.  Axis has all sorts of ways you can cut down on the noise with motion detection - these cameras are so sensitive I'm pretty sure they can detect a bee fart.  So for instance - detection can look at only part of the video frame, or ignore objects that appear briefly, or that are small, or that are "swaying" (trees).  However, after endlessly tweaking these options I was still getting clips of shadows or the damn insects that swarm the camera at night.

![Axis motion config](/assets/2019_ha/motion_config.png)

But hey, its the future, the all powerful AI [robots that eat old people's medicine for fuel][7] can just tell me if there's a person in the clip or not.

Initially I wanted to use something like [ImageAI][8] to run the person detection locally.  This worked out great in testing on my laptop.  The [Video Object Detection][9] tutorial detailed how to process a video clip and get back information about the types of objects detected.  

![imageai detection example](/assets/2019_ha/imageai.png)

However, ImageAI proved to be a complete pain in the ass to get setup on the Pi3 (basically dependency hell, and one of the Pi's CDNs was flapping).  Its also a somewhat CPU/GPU intensive process and may have taken an inordinate amount of time on the Pi3's poor little CPU.

I decided to check out AWS [Rekognition][10] instead.  So rather than processing the clip locally, I'll upload it to S3 and use Rekognition's [labeling][11] feature to tell me if there's any humans in the clip.

Rekognition object labeling is dead simple - upload a clip, get a job ID, once that job is complete you get back an array of all the different types of objects detected.   Unfortunately for me (and Amazon) if you search for anything related to Rekognition you get back a ton of articles about cops using it to profile minorities and other dystopian headlines, instead of how to actually use it.  Amazon's [official docs][12] are decent but for some reason insist that you use several other AWS services along with Rekognition just to determine when your job is finished.  A friend's theory is this is to lock you into the ecosystem because now you're dependent on a half dozen services rather than just two.

Here's the simple Ruby script the automation uses to run detection on the clip:

{% highlight ruby %}
require 'aws-sdk-s3'
require 'aws-sdk-rekognition'

raise "Invalid file" unless Pathname.new(ARGV[0]).exist?

s3 = Aws::S3::Resource.new(region:'us-east-1')
rek = Aws::Rekognition::Resource.new(region:'us-east-1')
bucket_name = 'some_bucket'

s3.bucket(bucket_name).create
obj = s3.bucket(bucket_name).object(File.basename(ARGV[0]))
obj.upload_file(ARGV[0])

puts "#{ARGV[0]} uploaded"

rek_resp = rek.client.start_label_detection({
  video: { 
    s3_object: {
      bucket: bucket_name,
      name: File.basename(ARGV[0]),
    },
  },
  min_confidence: 85
})

puts "Detection job queued #{rek_resp.job_id}"

job_finished = false
seconds_running = 0

while job_finished == false  
  raise 'Timeout' if seconds_running > 300
  
  result = rek.client.get_label_detection({
    job_id: rek_resp.job_id
  })
  # main difference between amazon docs, just keep checking the status of the
  # rekognition job rather than using SQS and STS or whatever alphabet soup they 
  # have in the code sample
  
  if result.job_status == 'SUCCEEDED'
    detected_count = result.labels.select {|l| l.label.name == 'Person' }.length
    puts "\nFound #{detected_count} people objects in #{seconds_running} seconds"
    job_finished = true
    if detected_count > 0
      exit(0)
    else
      exit(1)
    end
  elsif result.job_status == 'IN_PROGRESS'
    print '.'
    sleep 1
    seconds_running += 1
  else
    raise "ERROR"
  end
end
{% endhighlight %}

Run `detect.rb ./somevideo.mp4` and we upload the clip to S3, give the S3 object to Rekognition, wait for the job to finish.  

Usually this process takes about 30 seconds but I've seen it take several minutes.  Tuning ImageAI I was able to get results in just a few seconds.   I wish Rekognition had more options for the label function, like to increase speed at the cost of accuracy. The `min_confidence` option is just 1-100% confidence threshold in order for some potential object type to be labeled.

The script makes use of exit codes to aid in processing on Node-RED.  We check for an exit code of 0 before sending the clip.  If the script times out, fails for some other reason, or no people are detected the exit code will be 1 and the automation will stop.

And thats it.  Several weeks of trial and error to basically get a clip of the mailman every day.

![Just the mailman](/assets/2019_ha/mailman.png)
___

[^1]: That is dorks like me who insist that its somehow better to endlessly tinker with this stuff rather than just buy a Ring doorbell
[^2]: Home Assistant has its own mobile app and notification setup but I couldn't get this working with video clips.  I also tried Pushbullet but again could not get video clips working.

[1]: https://blog.google/products/google-nest/helpful-home/
[2]: https://www.home-assistant.io/
[3]: https://nodered.org/
[4]: https://www.youtube.com/watch?v=ACmydtFDTGs
[5]: https://www.home-assistant.io/getting-started/presence-detection/
[6]: https://www.home-assistant.io/components/telegram/
[7]: https://www.nbc.com/saturday-night-live/video/old-glory-insurance/n10766
[8]: https://github.com/OlafenwaMoses/ImageAI
[9]: https://github.com/OlafenwaMoses/ImageAI/blob/master/imageai/Detection/VIDEO.md
[10]: https://aws.amazon.com/rekognition/
[11]: https://docs.aws.amazon.com/rekognition/latest/dg/API_DetectLabels.html
[12]: https://docs.aws.amazon.com/rekognition/latest/dg/labels-detect-labels-image.html
[13]: https://www.axis.com/en-us/products/network-cameras
[14]: https://www.ui.com/unifi-switching/unifi-switch-poe/
[15]: https://www.home-assistant.io/components/stream/
[16]: https://gist.github.com/neontuna/76fd630f64953b76895745b078baa1cd
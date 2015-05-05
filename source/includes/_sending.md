# Sending commands

## Start Job

```ruby
/**
 * $json_input: Contains the Job description
 * $job_id: You must generate it and save it somewhere.
**/
$CTComSDK->start_job($json_input, $job_id);
```

<aside class="notice">
Call this method to start a new Transcoding Job. This method will take your JSON payload and will send it to the 'input' SQS queue configured for your client.
</aside>

The JSON input you pass in parameter contains information about the input file you want to transcode, and an array of objects listing and describing the output files you want.

Each object in the output array, describe one output file. They could be of different types. For example you can generate an AUDIO file out of a VIDEO. Or a THUMB out of a VIDEO.

The most simple and obvious usage is VIDEO to VIDEO.

<b>See JSON input example at:</b><br>
<a href="https://github.com/sportarchive/CloudTranscode/tree/master/client_example/input_samples" target="_blank">https://github.com/sportarchive/CloudTranscode/tree/master/client_example/input_samples</a>

### Input description

```json
{
	"input_type": "VIDEO",
	"input_bucket": "ClientA-bucket-in",
	"input_file": "/input1/test1.mp4",
	"outputs": 
	[
```
This describe the JSON field describing the input file.

key | type | default | mandatory | description | values
--- | ---- | ------- | --------- | ----------- | ------ 
input_type | string | none | yes | Type of the input file to transcode | VIDEO, AUDIO, DOC, IMG
input_bucket | string | none | yes | S3 bucket name | any
input_file | string | none | yes | filename of the file to download | any
job_label | string | job_id | no | Give a name to your job | any
outputs | array | none | yes | List of all the outputs needed for that job | array of max 100 entries

### Common Output description

These are the common JSON fields describing any output files of any types.

The output_type field is dependent on the input_type field. You cannot convert a VIDEO into a DOC for example.

key | type | default | mandatory | description | values
--- | ---- | ------- | --------- | ----------- | ------ 
output_type | string | none | yes | Type of the file to ouput | VIDEO, THUMB, AUDIO, DOC, IMG
output_bucket | string | none | yes | S3 bucket where the ouput file will be uploaded | any
output_file | string | none | yes | Path and filename of the file to generate and upload | any
s3_rrs | string | false | no | Activate Reduced redundancy or not in S3 storage | true, false
s3_encrypt | string | false | no | Activate backend storage encryption | true,false

### Video Output description
```json
{
...
    "outputs": [
        {
            "output_type": "VIDEO",
            "output_bucket": "ClientA-bucket-out",
            "output_file": "/output1/test1_sd.mp4",
            "s3_rrs": true,
            "s3_encrypt": true,
            "preset": {
                  "name": "System preset: Generic iphone4S",
                  "description": "Generic transcoding preset for: iPhone 4s and above, iPad 3G and above, iPad mini, Samsung Galaxy S2/S3/Tab 2",
                  "size": "1920x1080",
                  "frame_rate": 30,
                  "video_bitrate": "5000k",
                  "audio_bitrate": "160k",
                  "video_codec": "libx264",
                  "audio_codec": "libfdk_aac",
                  "video_codec_options": "MaxReferenceFrames:3,Profile:high,Level:4.1"
            },
            "watermark": {
                  "input_bucket": "ClientA-bucket-in",
                  "input_file": "/watermark/logo.png",
                  "size": "75.2x28.4",
                  "opacity": 0.2,
                  "x": -20,
                  "y": -20
            }
        }
    ]
}
```

There are the JSON fields describing VIDEO output files.

key | type | default | mandatory | description | values
--- | ---- | ------- | --------- | ----------- | ------
preset | object | none | yes | Preset to use to transcode the file | <a href="https://github.com/sportarchive/CloudTranscode-FFMpeg-presets" target="_blank">Find presets</a>
size | string | original size | no | Define output size. Valid for videos and images only | <width>x<height>
video_codec | string | from preset | no | Override the video codec used by preset | ffmpeg codec
audio_codec | string | from preset | no | Override the audio codec used by preset | ffmpeg codec
video_bitrate | string | from preset | no | Override video bitrate used by preset | valid bitrate
audio_bitrate | string | from preset | no | Override audio bitrate used by preset | valid bitrate
frame_rate | string | from preset | no | Override framerate used by preset | valid framerate
watermark | object | none | no | Describe watermark for the video | see <a href="#watermark">Watermark</a> section
keep_ratio | boolean | true | no | Keep the ratio of the original video. 4:3 or 16:9 | true,false
no_enlarge | boolean | true | no | Prevent enlarging the video size from original, even if bigger size is provided. | true,false

#### Watermark object
```json
{
...
    "watermark": {
        "input_bucket": "ClientA-bucket-in",
        "input_file": "/watermark/logo.png",
        "size": "75.2x28.4",
        "opacity": 0.2,
        "x": -20,
        "y": -20
    }
...
}
```

Use this object within a VIDEO output description to overlay an image on top of the video.

You can control many aspects of that image using the options below.

key | type | default | mandatory | description | values
--- | ---- | ------- | --------- | ----------- | ------
input_butcket | string | none | yes | Bucket name where the watermark is located | any
input_file |string |none |yes |Filename and path of the file containing the watermark |any
size |string |file default size |no |Size of the watermark in the video result |any
opacity |string |0.7 |no |Change the default watermark opacity |0.1 to 1.0
x |string |-10 |no |Change the watermark position. In pixels. |-0 to -n, 0 to n
y |string |-10 |no |Change the watermark position. In pixels. |-0 to -n, 0 to n

### Thumb Output description
> Generates a thumbnail from a precise location in the video (in seconds)

```json
{
...
    "outputs": [
        {
	    "output_type": "THUMB",
            "mode": "snapshot",
            "output_bucket": "ClientA-bucket-out",
            "output_file": "/output1/thumbs/test1_thumb.jpg",
            "s3_rrs": true,
            "s3_encrypt": true,
            "size": "160x120",
            "snapshot_sec": 5
        }
    ]
}
```

> Generates N thumbnails at each intervals (in seconds)

```json
{
...
    "outputs": [
        {
            "output_type": "THUMB",
            "mode": "intervals",
            "output_bucket": "ClientA-bucket-out",
            "output_file": "/output1/thumbs_interval/test2_thumb.jpg",
            "s3_rrs": true,
            "s3_encrypt": true,
            "size": "160x120",
            "intervals": 5
        }
    ]
}
```

These are the output attributes you must provide to generate thumbnails from a video. Thumbnails are an output_type on its own.

key | type | default | mandatory | description | values
--- | ---- | ------- | --------- | ----------- | ------
mode |string |snapshot |no |Type of thumbnails generation needed |snapshot, intervals
output_bucket |string |none |yes |S3 bucket and path where the ouput thumbnails will be uploaded |any
output_file |string |none |yes |filename for your thumbnails.We will append sequence number |any
intervals |integer |10 |no |Override default interval in seconds. |any
snapshot_sec |interger |0 |no |Second in the video when to take the snapshot |any
size |string |none |yes |Size of the thumbnails |any

## Cancel Job

## Get Job List

## Get Job Status


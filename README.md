A Raspberry Pi script for recording RPICam or webcam files over a long time frame.

I created this script so that it would record video using a Raspberry Pi and could record for a long period of time. I only had access to RPi 3B+ models, and I was worried that it would not be able to save large video files quickly enough. Therefore, I set this up to only record video for a length of one minute. I found that it could save and start recording the next file easily.

I had this recording continuously for over 30 days. Unfortunately, I found out that this created close to 40,000 individual files, which upset most computers when I tried to open the USB hard drive I had saved these to. To combat this, I set up a system where it will record 600 files into its own folder and then repeats this process. This then enables easy access to the files without giving your computer a minor heart attack.

You can change the video quality and frame rate at your leisure. It does save files as .h264 files, which is apparently preferred by the Raspberry Pi. These are easily converted using another one of my repositories.


Enjoy!

``` python

####### Rapsberry PI record and save video files ######


### Imports ###

import picamera
import time
import os
import signal
import sys


### Main code ###

#€ Specify directory to save files. As its for a PI I had it saving to an external usb
usb_drive_base_path = '/media/rpi/usb/' 

def signal_handler(sig, frame):

    print('Stopping recording...')
    camera.stop_recording()
    camera.close()
    sys.exit(0)

def create_new_directory(base_path, dir_index):
    new_path = os.path.join(base_path, f"videoset_{dir_index}")
    if not os.path.exists(new_path):
        os.makedirs(new_path)
    return new_path

## This section sets a delay so that the first video begins recording at the start of the next nearest minute
def delay_until_next_minute():
    
    current_second = int(time.strftime("%S"))
    if current_second != 0:
        time.sleep(60 - current_second)

## This section sets up a folder and you can set how many video files it saves in it before it creates a new folder and repeats
signal.signal(signal.SIGINT, signal_handler)
current_dir_index = 1
current_dir_path = create_new_directory(usb_drive_base_path, current_dir_index)
file_count = 0  # Initialize the count of files in the current directory.
max_files_per_dir = 600  # Set the maximum number of files per directory.

## Video file settings
with picamera.PiCamera() as camera:
    camera.resolution = (640, 480)  # Video resolution.
    camera.framerate = 30  # Framerate for the recording.

    delay_until_next_minute()  # Delay the start of recording to align with the next minute.

    initial_filename = os.path.join(current_dir_path, 'video.h264')  # Define the initial file name.
    camera.start_recording(initial_filename)
    file_count += 1  

    while True:  
        time.sleep(60)  

        ## Check if the current directory has reached its max file capacity.
        if file_count >= max_files_per_dir:
            current_dir_index += 1  # Increment the directory index for a new directory.
            current_dir_path = create_new_directory(usb_drive_base_path, current_dir_index)  # Create the new directory.
            file_count = 0  # Reset the file count for the new directory.

        ## Generate a new filename based on the current timestamp.
        timestamp = time.strftime("%y_%m_%d_%H_%M_%S")
        filename = f"video_{timestamp}.h264"
        camera.split_recording(os.path.join(current_dir_path, filename))
        file_count += 1  


```

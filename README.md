import time
from datetime import datetime
import os
from moviepy.editor import VideoFileClip
import cv2

def check_len_video(file_name):
    data = cv2.VideoCapture(file_name)
    frames = data.get(cv2.CAP_PROP_FRAME_COUNT)
    fps = int(data.get(cv2.CAP_PROP_FPS))
    seconds = int(frames/fps)
    return seconds

def find_work(list_time_schedules, list_link_schedules):
    j = 0
    for i in range(len(list_link_schedules)):
        if current_time > list_time_schedules[j] and current_time < list_time_schedules[j + 1]:
            current = current_time.split(':', 3)
            time_schedule = list_time_schedules[j + 1].split(':', 3)
            total_current = int(current[0])*3600 + int(current[1])*60 + int(current[2])
            total_schedule = int(time_schedule[0])*3600 + int(time_schedule[1])*60 + int(time_schedule[2])
            global time_out
            time_out = total_schedule - total_current
            break
        else:
            j = j + 2
    return list_link_schedules[i], time_out

def check_type_folder(mypath):
    img_fm = (".tif", ".tiff", ".jpg", ".jpeg", ".gif", ".png", ".eps", 
          ".raw", ".cr2", ".nef", ".orf", ".sr2", ".bmp", ".ppm", ".heif")
    vid_fm = (".flv", ".avi", ".mp4", ".3gp", ".mov", ".webm", ".ogg", ".qt", ".avchd")
    aud_fm = (".flac", ".mp3", ".wav", ".wma", ".aac")
    media_fms = {"image": img_fm, "video": vid_fm, "audio": aud_fm}

    fns = lambda path, media : [fn for fn in os.listdir(path) if any(fn.lower().endswith(media_fms[media]) for ext in media_fms[media])]
    img_fns, vid_fns, aud_fns = fns(mypath, "image"), fns(mypath, "video"), fns(mypath, "audio")

    if len(vid_fns) > 0:
        check = 1
    else:
        check = 0
    
    return check

def play_pictures(link_file):
    print("Computer is playing pictures!")
    list_pictures = os.listdir(link_file)

    for i in range(len(list_pictures)):
        command = "start " + list_pictures[i]
        os.system(command)
        time.sleep(10)

def main():
    list_time_schedules = []
    list_link_schedules = []
    with open("schedule.txt", "r") as file:
        ar = file.readlines()

    with open("schedule.txt", "r") as file:
        for i in range(len(ar)):
            arr = file.readline()
            arrr = arr.split('-', 3)
            list_time_schedules.append(arrr[0])
            list_time_schedules.append(arrr[1])
            list_link_schedules.append(arrr[2])

    j = 0

    while True:
        now = datetime.now()
        global current_time
        current_time = now.strftime("%H:%M:%S")
        link_file, time_work = find_work(list_time_schedules, list_link_schedules)
        os.chdir(link_file)
        check = check_type_folder(link_file)
        if check == 1:
            print(total_current)
            print(total_schedule)
            print(time_work)
            print("Computer is playing videos!")
            list_videos = os.listdir(link_file)
            timer = []

            for i in range(len(list_videos)):
                timerr = check_len_video(list_videos[i])
                timer.append(timerr)

            link = []

            for i in range(len(list_videos)):
                command = "start " + list_videos[i]
                link.append(command)

            if time_work > timer[j]:
                os.system(link[j])
                time.sleep(timer[j])
                j = j + 1
                if j > len(timer):
                    j = 0
            else:
                os.system(link[j])
                time.sleep(time_work)
                os.system("taskkill /f /im VLC.exe")

        else:
            play_pictures(link_file)
        
main()

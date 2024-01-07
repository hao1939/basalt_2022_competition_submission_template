# Try MineRL on Azure AKS

## 1. Setup AKS cluster with GPU node

Just saving your time reading through the doc https://learn.microsoft.com/en-us/azure/aks/gpu-cluster.

Prerequisite:
1. `az cli`
1. `kubectl`
1. Azure Subscription with GPU quota.

Follow the steps below:
1. Update the file `env` as your need, then run `. ./env`.
1. Run the script `setup_aks.sh` to create a AKS cluster.

## 2. Create a Pod for debugging, watch the screen record

1. Create a debug Pod:

    `kubectl apply -f minerl.yaml`.

1. Jump into the pod:

    `kubectl exec -it $(kubectl get pod -l app=minerl -o jsonpath="{.items[0].metadata.name}") -- bash`

1. There is no diaplay attached, run it with xvfb-run:

    `xvfb-run --server-num 44 -e /dev/stdout -s "-ac -screen 0 640x360x24" python run.py`

1. Record the screen using ffmpeg:

    `ffmpeg -f x11grab -video_size 640x360 -framerate 25 -i :44 -c:v libx264 -pix_fmt yuv420p -crf 23 output.mp4`

1. Stop the recording, then download the `output.mp4`:

    `kubectl cp default/$(kubectl get pod -l app=minerl -o jsonpath="{.items[0].metadata.name}"):/home/aicrowd/output.mp4 output.mp4`

Now you can watch the screen record on `output.mp4`.

## 3. Or you may want to stream the screen

1. You can stream the screen over rtmp then watch using VLC player.
Stream the screen, see https://trac.ffmpeg.org/wiki/StreamingGuide:

    `ffmpeg -f x11grab -video_size 640x360 -framerate 25 -i :44 -c:v libx264 -preset fast -pix_fmt yuv420p -crf 23 -s 640x360 -threads 0 -bufsize 8000k -f flv -listen 1 rtmp://0.0.0.0:8888/stream`

1. Port-forward to localhost 8888:

    `kubectl port-forward $(kubectl get pod -l app=minerl -o jsonpath="{.items[0].metadata.name}") 8888:8888`

3. Now you can watch the screen record using VLC plyaer via `rtmp://127.0.0.1:8888/stream`

## 4. Or view the game on vnc

1. Start the v11vnc on the display :44

    `x11vnc -display :44 -listen 0.0.0.0 -rfbport 5901 -forever -shared -viewonly -passwd choice_password`

2. Connect to vnc on `ip:1`, use the passwd.


## Reference:
1. https://www.derekseaman.com/2023/10/home-assistant-proxmox-ve-8-0-quick-start-guide-2.html
2. https://pve.proxmox.com/wiki/Package_Repositories#sysadmin_no_subscription_repo
3. https://medium.com/@chamikaw/heres-how-to-avoid-ring-subscription-using-home-assistant-and-why-you-should-not-do-it-7130f4ff0c94
4. https://www.hacs.xyz/docs/use/download/download/#to-download-hacs
5. https://www.hacs.xyz/docs/use/configuration/basic/#to-set-up-the-hacs-integration
6. https://www.home-assistant.io/integrations/telegram



## Steps to install Proxmox
1. Prepare the bootable USB drive with the Proxmox image
2. Install from USB drive
3. Visit https://{proxmox_ip}:8006
4. In pve -> Updates -> Repositories -> disable the enterprise repositories (unless you have paid for subscriptions)
5. refer to https://pve.proxmox.com/wiki/Package_Repositories#sysadmin_no_subscription_repo to configure non-enterprise repositories for necessary updates, i.e., update the followings into /etc/apt/sources.list:
```
deb http://ftp.debian.org/debian bookworm main contrib
deb http://ftp.debian.org/debian bookworm-updates main contrib

# Proxmox VE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

# security updates
deb http://security.debian.org/debian-security bookworm-security main contrib
```
6. In Updates, perform necessary upgrades

## Steps to install HAOS
1. download ha image
```
wget https://github.com/home-assistant/operating-system/releases/download/13.1/haos_ova-13.1.qcow2.xz
```
2. uncompress the image & copy to LVM
```
unxz haos_ova-13.1.qcow2.xz
qm importdisk 100 haos_ova-13.1.qcow2 local-lvm
```
3. create ha vm in proxmox, with:
- preloaded keys => set to uncheck
- 2 cores
- others are default
4. In ha vm
- click hardware, there should be an unused disk, double click to edit, if using a SSD disk, check "discard" before "Add"
- attach disk to vm (e.g., id 100)
- in Options, update boot order to scsi0 just added
5. start vm
6. ha should be available at http://{ha_ip}:8123/
7. create an account, e.g., admin + admin_pw, user + user_pw
8. Select location -> Reading, UK
9. default not submit analytics
10. click Finish to continue

## Steps to install Ring integration
1. From devices -> add integration -> Ring
2. enter Ring user ID and password, and OTP from 2FA

## Steps to install HACS
1. refer to https://www.hacs.xyz/docs/use/download/download/#to-download-hacs
2. Add a repository via https://my.home-assistant.io/redirect/supervisor_addon/?addon=cb646a50_get&repository_url=https%3A%2F%2Fgithub.com%2Fhacs%2Faddons
3. Install HACS add-on to http://{ha_ip}:8123
4. Reboot HA
5. after restart, go to  http://{ha_ip}:8123
6. devices -> add integration
7. hacs is available now
8. Follow the steps in https://www.hacs.xyz/docs/use/configuration/basic/#to-set-up-the-hacs-integration
9. go to github
10. login and activate and authorize

## Steps to install Ring-MQTT
1. In HA, add add-on repository
2. install Mosquitto broker; set as auto start
3. install ring-mqtt
4. start the add-on, open webui to input user ID and password
5. add generic camera with (details should be available from ring-mqtt log):
stream_Source: rtsp://03cabcc9-ring-mqtt:8554/{id}_live
still_Image_URL":"https://homeassistant:8123{{ states.camera.front_door_snapshot.attributes.entity_picture }}
6. define a folder to save or access the video files; in configuration.yaml, add:
```
homeassistant:
  allowlist_external_dirs:
    - "/config/www/video"
  media_dirs:
    ring: "/config/www/video"
```
7. May then create an automation to save video when motion is detected:
```
action:
  - service: ring_doorbell_camera.record
    metadata: {}
    data:
      duration: 30
      filename: /config/www/video/ring{{ now().strftime("%Y%m%d%H%M%S") }}.mp4
    target:
      device_id: {device_id}
mode: single
```

## Steps for Telegram integration
1. create bot from @botfather -> e.g., ha_bot
2. get my user ID from @getidsbot ->  e.g., {ha_bot_id}
or using {api_token}:
```
curl -X GET https://api.telegram.org/bot{api_token}/getUpdates
```
3. Update configuration.yaml with the chat IDs
```
telegram_bot:
  - platform: polling
    api_key: "{api_token}"
    allowed_chat_ids:
      - {chat_id_1}
      - {chat_id_2}
      - {chat_id_3}
notify:
  - platform: telegram
    name: {user_1}
    chat_id: {chat_id_1}
  - platform: telegram
    name: {user_2}
    chat_id: {chat_id_2}
```
4. May do simple message test via "developer tools" -> "actions", e.g.,
```
action: notify.{user_1}
data:
  message: "Yay! A message from Home Assistant."
```
5. May test sending video attachment too:
```
action:
  action: notify.{user_1}
  data:
    title: Send a video
    message: "That's an example that sends a video."
    data:
      video:
        - file: /config/www/video/ring{yyyymmddhhmmss}.mp4
          caption: Latest Ring video
```
6. Sample automations may look like:
```
alias: Ring Motion Detection - Save Video
description: ""
triggers:
  - type: motion
    device_id: {device_id}
    entity_id: {entity_id}
    domain: binary_sensor
    trigger: device
conditions: []
actions:
  - action: camera.record
    metadata: {}
    data:
      duration: 30
      lookback: 0
      filename: /config/www/video/ring{{ now().strftime("%Y%m%d%H%M%S") }}.mp4
    target:
      device_id: {device_id}
  - action: notify.{gmail_profile}
    metadata: {}
    data:
      message: Ring Motion Detection - video saved
      title: Home Assistant - Ring Motion Detected!
      target: {target_email_address}
  - action: telegram_bot.send_message
    metadata: {}
    data:
      message: Ring Motion Detected
      title: Home Assistant - Ring Motion Detection
      target: {target_telegram_id}
    enabled: false
mode: single
```

```
alias:  Folder Watcher for Ring video files
description: ""
mode: single
triggers:
  - event_type: folder_watcher
    event_data:
      event_type: created
    trigger: event
conditions: []
actions:
  - delay:
      hours: 0
      minutes: 1
      seconds: 0
      milliseconds: 0
  - action: telegram_bot.send_video
    data:
      authentication: digest
      file: /config/www/video/{{ trigger.event.data.file }}
      caption: New ring video found ({{ trigger.event.data.file }})
      target: {target_telegram_id}
```
7. Avoid using special characters in telegram file names, e.g., ```_text_``` is interpreted as _text_








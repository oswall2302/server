import os
import re
import json
import glob

!sudo apt update &>/dev/null && echo "apt cache successfully updated" || echo "apt cache update failed, you might receive stale packages"

from google.colab import drive
drive.mount('/content/drive')
%cd "/content/drive/My Drive/Minecraft-server"
!ls

if os.path.isfile("colabconfig.json"):
  colabconfig = json.load(open("colabconfig.json"))
else:
  colabconfig = {"server_type": "generic"}
  json.dump(colabconfig, open("colabconfig.json",'w'))

version = colabconfig["server_version"]
if colabconfig["server_version"] < "1.17":
  !sudo apt-get purge openjdk* > /dev/null 2>&1
  !sudo apt-get install openjdk-8-jre-headless &>/dev/null && echo "Yay! Openjdk8 has been successfully installed." || echo "Failed to install OpenJdk8."
elif colabconfig["server_version"] >= "1.20.5":
  !sudo apt-get purge openjdk* > /dev/null 2>&1
  !sudo apt-get install openjdk-21-jre-headless &>/dev/null && echo "Yay! Openjdk21 has been successfully installed." || echo "Failed to install OpenJdk8."
else:
  !sudo apt-get purge openjdk* > /dev/null 2>&1
  !sudo apt-get install openjdk-17-jre-headless &>/dev/null && echo "Yay! Openjdk17 has been successfully installed." || echo "Failed to install OpenJdk17."

java_ver = !java -version 2>&1 | awk -F[\"\.] -v OFS=. 'NR==1{print $2}'
if java_ver[0] == "17" :
  print("Se esta utilizando JAVA 17 - You are using JAVA 17.")
elif java_ver[0] == "21" :
  print("Se esta utilizando JAVA 21 - You are using JAVA 21.")
else:
  print("Estas utilizando una versión inferior a JAVA 17: ", java_ver[0], ". You are using a downgraded java version. Minecraft 1.17+ might not work.")

jar_list = {'paper': 'server.jar', 'fabric': 'fabric-server-launch.jar', 'generic': 'server.jar', 'forge': 'forge.jar', 'vanilla': 'vanilla.jar', 'snapshot': 'snapshot.jar', 'mohist': 'mohist.jar'}
jar_name = jar_list[colabconfig["server_type"]]

if colabconfig["server_type"] == "paper":
  server_flags = "-XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:MaxGCPauseMillis=200 -XX:+UnlockExperimentalVMOptions -XX:+DisableExplicitGC -XX:+AlwaysPreTouch -XX:G1NewSizePercent=30 -XX:G1MaxNewSizePercent=40 -XX:G1HeapRegionSize=8M -XX:G1ReservePercent=20 -XX:G1HeapWastePercent=5 -XX:G1MixedGCCountTarget=4 -XX:InitiatingHeapOccupancyPercent=15 -XX:G1MixedGCLiveThresholdPercent=90 -XX:G1RSetUpdatingPauseTimePercent=5 -XX:SurvivorRatio=32 -XX:+PerfDisableSharedMem -XX:MaxTenuringThreshold=1 -Dusing.aikars.flags=https://mcflags.emc.gs -Daikars.new.flags=true"
else:
  server_flags = ""
memory_allocation = "-Xms8704M -Xmx8704M"

# Chose the tunnle service you want to use
# Available options: ngrok, playit
tunnel_service = "playit" # Si prefieres usar NGROK cámbialo aquí <--------
print("Usando", tunnel_service)

if tunnel_service == "playit":
  !pip -q install pyngrok
  from pyngrok import conf, ngrok

  # Ask for the ngrok authtoken
  print("Consigue tu authtoken de https://dashboard.ngrok.com/auth")
  import getpass

  # v - - - - - - - TOKEN  - - - - - - - v

  authtoken = ""  # <---- TOKEN NGROK (PON AQUÍ TU TOKEN [TIENE QUE ESTAR ENTRE LAS "COMILLAS"])

  ! ngrok authtoken $authtoken # login to ngrok

  # Sets default ngrok region

  # v - - - - - - - REGIONES DISPONIBLES  - - - - - - - v

  # ap - Asia/Pacific (Singapore)
  # au - Australia (Sydney)
  # eu - Europa (Frankfurt - Alemania)
  # in - India (Mumbai)
  # jp - Japan (Tokyo)
  # sa - America del sur (São Paulo - Brasil)
  # us - United States (Ohio)
  conf.get_default().region = 'us'  # <--- Cambia esto por la region que quieras (solo ngrok)

  # ^ - - - - - - - REGIONES DISPONIBLES  - - - - - - - ^

  # Connect to ngrok
  url = playit.connect(25565, 'tcp')
  print('La IP de tu servidor es ' + ((str(url).split('"')[1::2])[0]).replace('tcp://', ''))

elif tunnel_service == "playit":
  ! curl -SsL https://playit-cloud.github.io/ppa/key.gpg | sudo apt-key add -
  ! sudo curl -SsL -o /etc/apt/sources.list.d/playit-cloud.list https://playit-cloud.github.io/ppa/playit-cloud.list
  print('Iniciando servidor...')
  ! sudo apt update &>/dev/null && sudo apt install playit &>/dev/null && echo "Playit.gg instalado" || echo "Error al instalar playit"
  if colabconfig["server_type"] == "forge":
    version = colabconfig["server_version"]
    if colabconfig["server_version"] < "1.17":
      oldpathtoforge = glob.glob(f"/content/drive/My Drive/Minecraft-server/forge-{version}-*.jar")
      if oldpathtoforge:  # Check if the list is not empty
        path = oldpathtoforge[0]  # Get the first path from the list
        print(path)
        ! playit & java $memory_allocation -jar "{path}" nogui
      else:
        print("No forge universal jar found.")
    else:
      pathtoforge = glob.glob(f"/content/drive/My Drive/Minecraft-server/libraries/net/minecraftforge/forge/{version}-*/unix_*.txt")
      if pathtoforge:  # Check if the list is not empty
        path = pathtoforge[0]  # Get the first path from the list
        print(path)
        ! playit & java @user_jvm_args.txt "@{path}" "$@"
      else:
        print("No unix_args.txt found.")
  else:
    ! playit & java $memory_allocation $server_flags -jar $jar_name nogui

print('Iniciando servidor...')

if colabconfig["server_type"] == "forge":
  version = colabconfig["server_version"]
  if colabconfig["server_version"] < "1.17":
    oldpathtoforge = glob.glob(f"/content/drive/My Drive/Minecraft-server/forge-{version}-*.jar")
    if oldpathtoforge:  # Check if the list is not empty
      path = oldpathtoforge[0]  # Get the first path from the list
      print(path)
      !java $memory_allocation -jar "{path}" nogui
    else:
      print("No forge universal jar found.")
  else:
    pathtoforge = glob.glob(f"/content/drive/My Drive/Minecraft-server/libraries/net/minecraftforge/forge/{version}-*/unix_*.txt")
    if pathtoforge:  # Check if the list is not empty
      path = pathtoforge[0]  # Get the first path from the list
      print(path)
      !java @user_jvm_args.txt "@{path}" "$@"
    else:
      print("No unix_args.txt found.")
else:
 !java $memory_allocation $server_flags -jar $jar_name nogui

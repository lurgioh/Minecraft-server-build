# Pixelmon Server Setup (AMP + Playit.gg) — Complete End-to-End Notes (Single Section)

This is my complete, single-section write-up of everything I did to stand up a Pixelmon Minecraft server using CubeCoders AMP on Debian and expose it publicly with Playit.gg (no router changes). It covers the exact commands I ran, what each screen in AMP means, which Forge/Java versions are compatible, how I loaded Pixelmon, how I tuned spawn rates, how I set up backups, how I monitored temps and usage, how I fixed common errors, and how I made the tunnel reliable. You can paste this whole thing into a README and follow along line-by-line.

I started with a clean Debian install and gave the machine a LAN IP (mine was 192.168.1.236). I made sure I could SSH in from my main computer. On the server I installed AMP with:
sudo apt update && sudo apt install -y curl
curl -sSL https://getamp.sh | bash
Then I opened a browser to https://<server-ip>:8080 and logged into AMP. In the top-right gear (Settings) → Configuration → New Instance Defaults I pasted my AMP license key (the personal $10 license looks like AMP-XXXX-XXXX-XXXX-XXXX). If AMP ever complained that the license was missing when creating an instance, I fixed it via SSH like this:
sudo su -
ampinstmgr licenseupdate AMP-XXXX-XXXX-XXXX-XXXX
ampinstmgr --licensedefault AMP-XXXX-XXXX-XXXX-XXXX
systemctl restart ampinstmgr
Back in the AMP home page (Application Deployment System tile), I clicked Create Instance, chose Minecraft Java Edition, named the instance “Pixelmon”, left “Update and Start” enabled, and created it. Inside the instance, I went to Configuration → Server and Startup and changed Server Type to Forge, Release Stream = Stable. For Pixelmon Reforged I used Minecraft 1.16.5 Forge 36.2.40. I clicked Download/Update so AMP pulled Forge and generated the server files. If EULA popped up, I accepted it in Configuration → EULA and started again.

Important Java note: Forge 1.16.5 + Pixelmon 9.x requires Java 8–11 (Java 17 often works; Java 21 causes IllegalAccessError/“sun.security.util” problems). On Debian 13 (trixie) I installed Java 11 from Adoptium (Temurin):
sudo apt install -y wget gnupg apt-transport-https
wget -O- https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo gpg --dearmor -o /usr/share/keyrings/adoptium.gpg
echo "deb [signed-by=/usr/share/keyrings/adoptium.gpg] https://packages.adoptium.net/artifactory/deb trixie main" | sudo tee /etc/apt/sources.list.d/adoptium.list
sudo apt update
sudo apt install -y temurin-11-jre
I verified and copied the Java path with:
java -version
readlink -f $(which java)
It printed something like /usr/lib/jvm/temurin-11-jre-amd64/bin/java. In AMP → Configuration → Java and Memory I pasted that into “Java Path” so the server uses Java 11. Then I clicked Save, Stop, and Start.

To load Pixelmon I downloaded Pixelmon Reforged for 1.16.5 from https://reforged.gg (or CurseForge) on my main computer. In AMP’s File Manager I navigated to the instance’s minecraft/mods folder (if “mods” didn’t exist I created it) and uploaded the Pixelmon .jar (for example Pixelmon-1.16.5-9.1.10/9.1.11-universal.jar). Back in Status/Console I started the server and watched logs; I expected to see Forge initialize and Pixelmon register content without red stacktraces. Typical first boot CPU/memory spikes are normal because Forge unpacks lots of assets; after “Done (X.XXs)! For help, type "help" or "?"” CPU falls back. If the console ever stopped with Java errors, I double-checked the Java version and the Forge build.

Client side, to join I installed Forge 1.16.5 on my PC, created/used the Forge profile in the Minecraft launcher, and dropped the same Pixelmon .jar into my local .minecraft/mods folder. On LAN I could connect to 192.168.1.236:25565. For friends over the internet, I used Playit.gg so I didn’t need port forwarding. I installed Playit, linked the agent, and created a tunnel:
sudo apt install -y playit
playit
The first run prints “Visit link to setup https://playit.gg/claim/XXXX”. I opened that claim URL in my browser while logged into Playit. In the Playit dashboard I created a tunnel: Protocol = TCP, Local Address = 192.168.1.236, Local Port = 25565, Name = Minecraft Java. Playit gave me a public address like national-kenneth.na.joinmc.link (the exact host varies per region/account); while the playit process runs, that hostname forwards to my server. If the dashboard showed my agent offline or kept asking to claim even after I claimed, I reset the agent’s local state with:
rm -rf ~/.playit
playit
and re-claimed the new link. The terminal then printed “Connected to Playit network” and showed the active tunnel mapping to 192.168.1.236:25565. In Minecraft I used that playit host:port in Multiplayer → Add Server and connected from anywhere.

For spawn rates I tuned Pixelmon’s config. In AMP → File Manager I opened config/pixelmon/spawning.yml. Under the global section I raised the base spawn frequency:
global:
  spawnRate: 2.0
I saved the file and restarted the server; 2.0 roughly doubles spawns vs. default 1.0. I avoided extreme values (like 5+) because they can cause lag and memory pressure. For testing, I used the in-game command /checkspawns to see what should spawn in the current biome/time. If despawns felt too aggressive, I increased the time Pokémon stick around (names can vary by version; the idea is to lengthen despawn timing in the same spawning config).

For backups I used AMP’s built-in tool so I could safely experiment. Before shutdowns or big mod changes I went to Backups and clicked Create Backup; AMP zipped the entire instance (world, configs, mods). I kept copies somewhere safe (NAS or another disk). Restores are simple inside AMP: choose a backup zip and click Restore. If I wanted scheduled backups later, I added a Scheduled Task → Perform Backup nightly and set retention to keep the last few zips.

To keep an eye on temps on Debian I installed lm-sensors and scanned for chips:
sudo apt install -y lm-sensors
sudo sensors-detect
sensors
This printed CPU package/core temps. For a live view I used:
watch -n 2 sensors
If I wanted a visual dashboard I installed Netdata:
bash <(curl -Ss https://my-netdata.io/kickstart.sh)
then opened http://192.168.1.236:19999 to see CPU, temps (when supported by the board), RAM, disk, and network graphs. Normal behavior on first server boot is transient high CPU (30–80%) and ~2–4 GB RAM use while Forge and Pixelmon initialize; it should settle once the world is running. If CPU stayed high at idle, I checked the console for repeating errors, reduced view distance, and verified Java 11/17 (not 21) was selected in AMP.

Troubleshooting I ran into and fixes that worked for me:
• “Internal Exception: java.io.IOException: An existing connection was forcibly closed by the remote host” when joining via Playit: usually the tunnel or server wasn’t fully online. I made sure playit showed “Connected” and that the Playit tunnel pointed to local port 25565. I also waited for the server to show “Done” before connecting.  
• Java 21 crash/IllegalAccessError: switch the instance to Java 11 (Temurin) by installing temurin-11-jre and setting the Java Path in AMP to /usr/lib/jvm/temurin-11-jre-amd64/bin/java. Stop/Start the instance.  
• No “mods” folder at first: start the Forge server once (it generates folder structure), then stop; or create minecraft/mods manually.  
• AMP says “license key must be specified”: set the license under ADS (home) Configuration → New Instance Defaults, or run ampinstmgr licenseupdate … and ampinstmgr --licensedefault … then systemctl restart ampinstmgr.  
• Pixelmon not spawning enough: raise global.spawnRate in config/pixelmon/spawning.yml, restart, and test with /checkspawns.  
• Memory/lag bursts: in AMP → Configuration → Java and Memory I set Initial (Xms) to 2G and Maximum (Xmx) to 4–6G (not more than ~75% of system RAM). I avoided huge view distances.  
• Playit agent keeps asking to claim: rm -rf ~/.playit, run playit again, claim the new link in the browser, confirm the agent shows “Online” in the dashboard.  
• Verifying the tunnel from terminal: playit prints “tunnel running, 1 tunnels registered” and lists something like national-kenneth.na.joinmc.link => 192.168.1.236:25565 (minecraft-java).

Finally, a quick operational flow that worked well for me: (1) start AMP and the instance; (2) wait for “Done”; (3) run playit (or keep it running in a screen/tmux or service); (4) share the playit hostname with friends; (5) when finished, in AMP create a backup, then stop the instance cleanly and shut down the box if needed. This whole setup let me run a stable Pixelmon Reforged server on Debian, fully managed in a browser via AMP, and reachable globally through a Playit.gg hostname with no router changes.

— Harrison Lurgio (MIS + Cybersecurity, FAU) — GitHub: https://github.com/harrisonlurgio

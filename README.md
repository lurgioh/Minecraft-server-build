# 🎮 Pixelmon Minecraft Server (AMP + Playit.gg) – Debian 13 Build

📘 **Project Overview**  
This project documents the full setup, configuration, and optimization process for hosting a **Pixelmon Reforged Minecraft server** using **CubeCoders AMP** for management and **Playit.gg** for secure, public connectivity — no port forwarding required.  
The goal was to create a reliable, easily managed server environment on a Debian 13 system, capable of supporting modded gameplay with monitoring, backups, and external access.

---

🧠 **System Specs**

| Component | Details |
|------------|----------|
| **CPU** | Intel Core i5-7500 (4 Cores / 4 Threads) |
| **RAM** | 8 GB DDR4 |
| **Storage** | 128 GB SSD |
| **Operating System** | Debian 13 (Trixie) – Minimal / Headless |
| **Network** | Gigabit LAN |
| **Server Purpose** | Pixelmon Reforged Modded Minecraft Server |

---

⚙️ **Setup Process**

1. **OS Preparation**  
   - Installed Debian 13 headless with SSH enabled.  
   - Set static IP: `192.168.1.236`.  
   - Updated packages using `sudo apt update && sudo apt upgrade -y`.

2. **Installed AMP (Application Management Panel)**  
   - Installed via script:  
     ```bash
     curl -sSL https://getamp.sh | bash
     ```  
   - Accessed AMP dashboard at `https://192.168.1.236:8080`.  
   - Activated license and created a new **Minecraft Java (Forge)** instance.  
   - Selected **Forge Version 36.2.40 (Minecraft 1.16.5)**.  

3. **Configured Java 11 (Temurin)**  
   - Installed Java 11:  
     ```bash
     sudo apt install wget gnupg apt-transport-https -y
     ```  
   - Added Adoptium repo and installed `temurin-11-jre`.  
   - Set Java path in AMP to `/usr/lib/jvm/temurin-11-jre-amd64/bin/java`.

4. **Installed Pixelmon Mod**  
   - Downloaded Pixelmon Reforged (1.16.5) from [https://reforged.gg](https://reforged.gg).  
   - Uploaded the `.jar` file (e.g., `Pixelmon-1.16.5-9.1.11-universal.jar`) to AMP → `/mods`.  
   - Restarted instance and confirmed `[Pixelmon] Pixelmon Reforged initializing...`.

5. **Configured Playit.gg (Public Access)**  
   - Installed Playit:  
     ```bash
     sudo apt install playit -y
     playit
     ```  
   - Claimed the tunnel and linked it to the server (TCP → `192.168.1.236:25565`).  
   - Received a public domain like `yourname.na.joinmc.link`.  
   - Verified tunnel was active and reachable globally.

6. **Spawn Rate & Optimization Tweaks**  
   - Modified `config/pixelmon/spawning.yml`  
     ```yaml
     global:
       spawnRate: 2.0
     ```  
   - Restarted server and confirmed higher spawn frequency via `/checkspawns`.

7. **Backups & Automation**  
   - Used AMP’s built-in Backups → “Create New Backup.”  
   - Created a recurring backup schedule under Configuration → Scheduled Tasks.  
   - Manual backups stored at `/home/harrison/backups/`.

8. **System Monitoring**  
   - Installed `lm-sensors` and `Netdata` for hardware and performance tracking.  
     - `sudo apt install lm-sensors -y`  
     - `bash <(curl -Ss https://my-netdata.io/kickstart.sh)`  
   - Monitored at `http://192.168.1.236:19999`.

---

🔧 **Issues & Troubleshooting**

- **Issue:** Playit tunnel kept asking to claim again.  
  **Fix:** Removed old data with `rm -rf ~/.playit` and reconnected.  

- **Issue:** Java errors or crash loops.  
  **Fix:** Verified AMP instance was using Java 11 (Temurin).  

- **Issue:** Missing mod dependencies.  
  **Fix:** Confirmed Pixelmon `.jar` was uploaded to `/mods`.  

- **Issue:** Low spawn rates.  
  **Fix:** Increased global `spawnRate` to 2.0 in Pixelmon config.  

- **Issue:** Overheating under load.  
  **Fix:** Installed `lm-sensors`, confirmed CPU temps stable (~45°C).  

---

🚀 **Final Configuration**

- Debian 13 headless environment  
- AMP-managed Minecraft Forge server  
- Pixelmon Reforged 1.16.5  
- Playit.gg public tunnel  
- Automated backups and monitoring  
- Secure web management via AMP dashboard  

---

🧠 **Key Takeaways**

- Learned advanced AMP configuration and modded server management.  
- Implemented secure tunneling via Playit.gg without router access.  
- Gained experience in headless Debian setup and performance monitoring.  
- Practiced network troubleshooting and remote access control.  
- Improved Linux administration and system automation skills.  

---

📈 **Future Improvements**

- Add scheduled reboots and alert notifications in AMP.  
- Integrate Grafana dashboards for long-term monitoring.  
- Migrate backups to offsite or NAS storage.  
- Experiment with modpacks and plugin isolation testing.  

---

🏁 **Final Thoughts**

This project combined Linux system administration, network tunneling, and modded server management into one cohesive build.  
From installation to optimization, it showcased the power of AMP and Playit.gg for managing complex game environments without router access.  
The end result is a reliable, secure, and fully-automated Pixelmon server that runs efficiently on Debian — perfect for long-term multiplayer hosting and hands-on experimentation.

---

✨ **Created by Harrison Lurgio**  
MIS + Cybersecurity Student @ Florida Atlantic University  
🔗 GitHub: [https://github.com/lurgioh](https://github.com/lurgioh)

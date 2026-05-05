# Offensive/Defensive Hybrid VM — REMnux in Docker on Kali Linux

Today I set up a Docker container inside my Kali Linux VM running REMnux, creating a single environment with both offensive (Kali) and defensive/malware analysis (REMnux) tooling.

---

## Objective

Deploy REMnux as an isolated Docker container inside Kali Linux, with a shared volume for passing malware samples between environments — combining red team and blue team tooling without running two separate VMs.

---

## Tools Used

- **Kali Linux** — Penetration testing host OS
- **Docker** — Container runtime
- **REMnux** (remnux/remnux-distro:focal) — Linux malware analysis distro
- **Docker Volume Mount** — Shared folder between Kali host and REMnux container

---

## Steps

### 1. Update Kali

```bash
sudo apt update && sudo apt upgrade -y
```

![Kali Updated](image/kali-updated.png)

---

### 2. Install and Start Docker

```bash
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER && newgrp docker
sudo systemctl status docker
```

![Docker Active](image/docker-active.png)

---

### 3. Pull the REMnux Image

```bash
docker pull remnux/remnux-distro:focal
docker images
```

![REMnux Pulled](image/remnux-pulled.png)

---

### 4. Create Shared Sample Folder and Launch Container

```bash
mkdir ~/Desktop/REMnuxSamples

docker run -it --name remnux \
  -v ~/Desktop/REMnuxSamples:/home/remnux/samples \
  remnux/remnux-distro:focal bash
```

![REMnux Container Launched](image/remnux-container-launched.png)

---

### 5. Verify REMnux Tools

Inside the container:

```bash
strings --version
python3 --version
floss --help 2>&1 | head -5
```

![REMnux Tools Verified](image/remnux-tools-verified.png)

---

### 6. Exit and Manage the Container

```bash
exit
docker ps -a
```

![Docker PS Exited](image/docker-ps-exited.png)

---

### 7. Restart the Container

```bash
docker start -ai remnux
```

![Docker Restarted](image/docker-restarted.png)

---

### 8. Confirm Shared Volume

Inside the container:

```bash
touch /home/remnux/samples/test.txt
```

On Kali:

```bash
ls ~/Desktop/REMnuxSamples
```

![Shared Volume Confirmed](image/shared-volume-confirmed.png)

---

## Troubleshooting

**Corrupted ZSH History**

While adding the user to the Docker group, the following error appeared:
`zsh: corrupt history file /home/kali/.zsh_history`

This is unrelated to Docker — the history file simply became corrupted. Fixed with:

```bash
mv .zsh_history .zsh_history.bak
touch .zsh_history
```

Docker was unaffected and continued running normally.

---

## Workflow

1. Drop a sample into `~/Desktop/REMnuxSamples/` on Kali
2. `docker start -ai remnux`
3. `cd /home/remnux/samples` — file is available
4. Run static analysis tools against it
5. Output files are accessible back in `~/Desktop/REMnuxSamples` on Kali

---

## Conclusion

Successfully deployed REMnux inside a Docker container on Kali Linux with a shared volume for sample analysis. This setup gives a single VM the capability to perform both offensive security operations and blue team malware analysis without the overhead of running two full virtual machines.

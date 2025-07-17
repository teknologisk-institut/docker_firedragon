# Building, Uploading, and Using a Docker Image with GitHub Container Registry (GHCR)

## 1. **Prepare your project**

Organize your files like this on your computer:

```
project-root/
│
├── Dockerfile
├── src/
│   └── (your private ROS2 and custom packages, etc.)
```
---

## 2. **Build the Docker Image**

From your `project-root` directory:

```bash
docker build -t firedragon:latest .
```

---

## 3. **Tag for GHCR**

If your organization is `teknologisk-institut`, tag the image like:

```bash
docker tag firedragon:latest ghcr.io/teknologisk-institut/firedragon:latest
```

---

## 4. **Create a GitHub Personal Access Token (PAT)**
- Go to [GitHub PAT settings](https://github.com/settings/tokens/new)
- Add scopes: `write:packages` and `read:packages`
- If using an organization, enable SSO if needed
- Copy the token securely

---

## 5. **Login to GHCR**

```bash
echo YOUR_GITHUB_PAT | docker login ghcr.io -u GITHUB_USERNAME --password-stdin
```

---

## 6. **Push to GHCR**

```bash
docker push ghcr.io/teknologisk-institut/firedragon:latest
```

---

## 7. **(Optional) Make Package Public**

By default, GHCR images are **private**.  
To make it public:

1. Go to [https://github.com/orgs/teknologisk-institut/packages](https://github.com/orgs/teknologisk-institut/packages)
2. Click your image (`firedragon`)
3. Click **Package Settings**
4. Change visibility from **Private** to **Public** [[GitHub Docs](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)]

---

## 8. **Test: Pull and Run the Image (on another machine or by your team)**

### **If Public**
Anyone can pull _without_ logging in:

```bash
docker pull ghcr.io/teknologisk-institut/firedragon:latest
```

### **If Private**
Login first with your PAT:

```bash
echo YOUR_GITHUB_PAT | docker login ghcr.io -u GITHUB_USERNAME --password-stdin
docker pull ghcr.io/teknologisk-institut/firedragon:latest
```

---

## 9. **Setup Livox avia lidar**
Set a wired connection on the host machine with the following paramters.
IP:             10.46.28.110
Subnet Mask:    255.255.255.0
Gateway:        10.46.28.1

Our lidar (DTI) has the following IP: 10.46.28.111

## 10. **Run the Container (with hardware/network and GUI access)**

```bash
xhost +local:root
docker run --rm -it \
    --network=host \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    ghcr.io/teknologisk-institut/firedragon:latest
```

---

## 11. **Inside the Container**

Source ROS2 and your workspace:

```bash
source /opt/ros/humble/setup.bash
source /firedragon_ws/install/setup.bash
```
Then launch your ROS nodes, e.g.:
```bash
ros2 launch livox_ros2_driver livox_lidar_rviz_launch.py
```
---

## **Troubleshooting**

- **permission_denied: create_package**: Make sure you are pushing to the correct org (`ghcr.io/orgname/imagetag`) and have the right PAT scopes and permissions (see steps above).
- **Can't pull?** Check if image is public. If not, login first with a PAT with at least `read:packages`.
- **Org collaboration:** Members need `read:packages` (pull) or `write:packages` (push) permissions for the package.

---

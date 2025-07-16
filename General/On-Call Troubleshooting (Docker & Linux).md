# Docker & Linux On-Call Troubleshooting and Scenario-Based Interview Questions

---

## 1. Docker Container Management

1. **Container fails to start: "OCI runtime create failed: no such file or directory: /bin/bash"**
   - **Solution**: Image lacks `/bin/bash` or incorrect entrypoint.
     ```bash
     docker inspect <image_name> | grep -i entrypoint
     docker run -it <image_name> /bin/sh -c "ls /bin/bash"
     ```
     - Install bash: `RUN apt-get update && apt-get install -y bash`
     - Or use `/bin/sh`: `docker run -it --entrypoint /bin/sh <image_name>`
   - **Considerations**: Ensure base image has required shell; minimal images (e.g., alpine) may need `sh`.

2. **Container exits with code 137**
   - **Solution**: Out-of-memory (OOM) error.
     ```bash
     docker logs <container_id>
     docker inspect <container_id> | grep -i memory
     docker run --memory="512m" <image_name>
     ```
   - **Considerations**: Check host memory (`free -m`), application leaks, or adjust `--memory-swap`.

3. **Container stuck in "Created" state**
   - **Solution**:
     ```bash
     docker ps -a
     docker inspect <container_id> | jq '.State.Error'
     journalctl -u docker.service
     docker start <container_id>
     ```
     - Recreate if needed: `docker rm <container_id> && docker run <image_name>`
   - **Considerations**: Verify Docker daemon (`systemctl status docker`), resources, and architecture compatibility.

4. **Application in container not responding**
   - **Solution**:
     ```bash
     docker exec -it <container_id> /bin/sh
     ps aux
     docker logs <container_id>
     curl http://localhost:<port>
     docker stats <container_id>
     ```
   - **Considerations**: Check application logs, port binding, and avoid stopping container during debugging.

5. **Container uses excessive CPU**
   - **Solution**:
     ```bash
     docker stats <container_id>
     docker exec <container_id> top
     docker update --cpus="0.5" <container_id>
     ```
   - **Considerations**: Set CPU limits (`--cpus`), check application loops, and monitor host CPU (`htop`).

6. **Container not accessible, possibly not running**
   - **Solution**:
     ```bash
     docker ps -a | grep <container_name>
     docker start <container_id>
     docker logs <container_id>
     ```
     - Recreate: `docker rm <container_id> && docker run <image_name>`
   - **Considerations**: Verify configuration, Docker daemon, and host resources (`df -h`, `free -m`).

7. **Recreate accidentally deleted container**
   - **Solution**:
     ```bash
     docker inspect <image_name> | grep -i cmd
     docker run -d --name <container_name> -p <host_port>:<container_port> -v <volume_path> <image_name>
     docker-compose up -d
     ```
   - **Considerations**: Store configurations in Docker Compose/scripts, verify volumes, and check environment variables.

8. **Scale Dockerized application**
   - **Solution**:
     ```bash
     docker-compose up -d --scale <service_name>=3
     docker swarm init
     docker service create --replicas 3 --name <service_name> <image_name>
     docker run -d -p 80:80 nginx
     ```
     - NGINX config:
       ```nginx
       upstream app {
           server container1:8080;
           server container2:8080;
           server container3:8080;
       }
       server { listen 80; location / { proxy_pass http://app; } }
       ```
   - **Considerations**: Ensure stateless application, monitor resources (`docker stats`), use external load balancer for production.

9. **Container fails: "cannot allocate memory"**
   - **Solution**:
     ```bash
     free -m
     docker inspect <container_id> | grep -i memory
     docker run --memory="1g" <image_name>
     docker system prune
     ```
   - **Considerations**: Check for memory leaks, ensure host memory, adjust swap settings.

10. **Update container to latest application version**
    - **Solution**:
      ```bash
      docker pull <image_name>:latest
      docker-compose up -d
      docker service update --image <image_name>:latest <service_name>
      docker exec <container_id> <app_version_command>
      ```
    - **Considerations**: Test in staging, ensure data compatibility, use health checks.

---

## 2. Docker Networking

11. **Container cannot connect to external database**
    - **Solution**:
      ```bash
      docker inspect <container_id> | grep -i network
      docker exec <container_id> ping <db_host>
      docker exec <container_id> nslookup <db_host>
      sudo iptables -L -n
      sudo iptables -A INPUT -p tcp --dport <db_port> -j ACCEPT
      ```
    - **Considerations**: Verify database host/port, network mode, and firewall/security groups.

12. **Containers in same network cannot communicate**
    - **Solution**:
      ```bash
      docker network ls
      docker inspect <container_id> | grep -i network
      docker exec <container1_id> ping <container2_name>
      docker network create mynet
      docker network connect mynet <container_id>
      ```
      - Docker Compose:
        ```yaml
        version: '3'
        services:
          app1: { image: <image1>, networks: [mynet] }
          app2: { image: <image2>, networks: [mynet] }
        networks: { mynet: { driver: bridge } }
        ```
    - **Considerations**: Use container names for DNS, check network policies, restart after changes.

13. **Containerized app not accessible via mapped port**
    - **Solution**:
      ```bash
      docker inspect <container_id> | grep -i port
      docker exec <container_id> netstat -tuln
      curl http://localhost:<host_port>
      sudo iptables -L -n
      sudo iptables -A INPUT -p tcp --dport <host_port> -j ACCEPT
      ```
    - **Considerations**: Verify port mapping, application binding (`0.0.0.0`), and conflicting host services.

14. **Port conflict: "Bind for 0.0.0.0:80 failed"**
    - **Solution**:
      ```bash
      sudo netstat -tulnp | grep :80
      sudo kill <pid>
      docker run -p 8080:80 <image_name>
      ```
    - **Considerations**: Check for conflicting containers (`docker ps`), avoid duplicate port usage.

15. **Container cannot resolve DNS names**
    - **Solution**:
      ```bash
      docker exec <container_id> nslookup google.com
      docker inspect <container_id> | grep -i dns
      docker run --dns 8.8.8.8 <image_name>
      cat /etc/resolv.conf
      ```
    - **Considerations**: Verify host DNS, check network policies (port 53), restart after DNS changes.

16. **Container dropping packets intermittently**
    - **Solution**:
      ```bash
      docker inspect <container_id> | grep -i network
      docker exec <container_id> tcpdump -i eth0
      sudo tcpdump -i docker0
      docker stats <container_id>
      ```
    - **Considerations**: Check for congestion, MTU settings, or network driver issues (`ifconfig`).

17. **Restrict network traffic between containers**
    - **Solution**:
      ```bash
      docker network create --driver bridge --opt com.docker.network.bridge.enable_icc=false mynet
      docker network connect mynet <container_id>
      sudo iptables -A FORWARD -s <container1_ip> -d <container2_ip> -p tcp --dport <port> -j ACCEPT
      ```
    - **Considerations**: Test connectivity, use Docker Compose networks, avoid blocking critical traffic.

18. **Containerized web server returns 502 Bad Gateway**
    - **Solution**:
      ```bash
      docker logs <container_id>
      docker exec <web_container_id> curl http://<backend_service>:<port>
      docker inspect <container_id> | grep -i network
      ```
      - NGINX config:
        ```nginx
        server { listen 80; location / { proxy_pass http://<backend_service>:<port>; } }
        ```
    - **Considerations**: Verify backend health, network configuration, and proxy settings.

19. **Slow container network performance**
    - **Solution**:
      ```bash
      docker stats <container_id>
      docker run -it --rm <image_name> iperf -c <target_ip>
      sudo iftop -i docker0
      docker network inspect <network_name> | grep -i mtu
      ```
    - **Considerations**: Compare with other containers, check congestion or QoS, verify network driver.

20. **Container blocked from external API by firewall**
    - **Solution**:
      ```bash
      sudo iptables -L -n -v
      docker exec <container_id> curl <api_url>
      sudo iptables -A OUTPUT -p tcp --dport <api_port> -j ACCEPT
      ip route
      ```
    - **Considerations**: Verify firewall rules, cloud security groups, and test connectivity.

---

## 3. Docker Storage and Volumes

21. **Container cannot write to mounted volume**
    - **Solution**:
      ```bash
      ls -ld /path/to/volume
      chmod -R 775 /path/to/volume
      chown -R user:group /path/to/volume
      docker inspect <container_id> | jq '.Mounts'
      setenforce 0
      docker restart <container_id>
      ```
    - **Considerations**: Align host/container permissions, check SELinux/AppArmor.

22. **Volume filling up host disk**
    - **Solution**:
      ```bash
      du -sh /var/lib/docker/volumes/*
      docker stop <container_id>
      docker run --rm -v <volume_name>:/data busybox sh -c "rm -rf /data/unneeded_files"
      docker volume prune
      ```
    - **Considerations**: Stop containers before cleanup, verify safe file deletion, use pruning cautiously.

23. **Corrupted volume recovery**
    - **Solution**:
      ```bash
      docker volume inspect <volume_name>
      docker run --rm -v <volume_name>:/data -it busybox sh
      fsck /path/to/volume
      tar -xzf /backup/volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>/_data
      docker volume create new_volume
      docker run -v new_volume:/app <image_name>
      ```
    - **Considerations**: Maintain backups, test restored data, use recovery containers.

24. **Volume error: "no such file or directory"**
    - **Solution**:
      ```bash
      docker volume ls
      ls -ld /path/to/host/mount
      mkdir -p /path/to/host/mount
      docker run -v /path/to/host/mount:/app <image_name>
      cat Dockerfile | grep VOLUME
      ```
    - **Considerations**: Ensure volume/path exists, check path spelling, verify Docker commands.

25. **Migrate volume to new host**
    - **Solution**:
      ```bash
      docker stop <container_id>
      tar -czf volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>/_data .
      scp volume_backup.tar.gz user@new_host:/tmp
      docker volume create <volume_name>
      tar -xzf /tmp/volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>/_data
      docker run -v <volume_name>:/app <image_name>
      ```
    - **Considerations**: Ensure Docker version compatibility, verify permissions, test post-migration.

26. **Bind mount changes not reflected on host**
    - **Solution**:
      ```bash
      docker inspect <container_id> | jq '.Mounts'
      ls -ld /path/to/host/mount
      chmod -R 775 /path/to/host/mount
      docker exec <container_id> id
      docker run -v /path/to/host/mount:/app:rw <image_name>
      chcon -R -t container_file_t /path/to/host/mount
      ```
    - **Considerations**: Confirm `rw` flag, align permissions, check SELinux.

27. **Unused volumes consuming disk space**
    - **Solution**:
      ```bash
      docker volume ls
      docker volume ls -f dangling=true
      docker volume prune
      df -h
      ```
    - **Considerations**: Verify volume dependencies, avoid accidental data loss.

28. **Expand database container storage**
    - **Solution**:
      ```bash
      docker exec <container_id> df -h
      docker volume create new_volume
      docker run --rm -v <old_volume>:/old -v new_volume:/new busybox sh -c "cp -a /old/. /new/"
      docker stop <container_id>
      docker run -v new_volume:/data <image_name>
      ```
    - **Considerations**: Minimize downtime, verify data integrity, ensure host disk space.

29. **Container cannot access NFS volume**
    - **Solution**:
      ```bash
      ping <nfs_server_ip>
      mount | grep nfs
      apt-get install -y nfs-common
      mount -t nfs <nfs_server_ip>:/share /mnt
      docker inspect <container_id> | jq '.Mounts'
      docker run -v <nfs_server_ip>:/share:/mnt <image_name>
      ```
    - **Considerations**: Verify NFS server, firewall, and client tools.

30. **Ensure data persistence across restarts**
    - **Solution**:
      ```bash
      docker volume create persistent_volume
      docker run -v persistent_volume:/app/data <image_name>
      docker inspect persistent_volume
      ```
    - **Considerations**: Use named volumes, back up critical data, verify mounts.

---

## 4. Docker Image Management

31. **Build fails: "COPY failed: no such file or directory"**
    - **Solution**:
      ```bash
      cat Dockerfile | grep COPY
      ls -l ./xyz
      ```
      - Fix: `COPY ./xyz /app/xyz`
      ```bash
      docker build -t my_image .
      ```
    - **Considerations**: Verify file paths, ensure files in build context.

32. **Optimize Dockerfile for faster builds**
    - **Solution**:
      ```dockerfile
      COPY requirements.txt .
      RUN pip install -r requirements.txt
      COPY . .
      ```
      - Multi-stage:
        ```dockerfile
        FROM node:16 AS builder
        WORKDIR /app
        COPY package.json .
        RUN npm install
        FROM node:16-slim
        COPY --from=builder /app /app
        ```
      ```bash
      docker build --cache-from my_image:latest -t my_image .
      ```
    - **Considerations**: Cache dependencies, use multi-stage builds, minimize copied files.

33. **Missing dependency in custom image**
    - **Solution**:
      ```bash
      docker logs <container_id>
      cat Dockerfile
      ```
      - Add: `RUN apt-get update && apt-get install -y <dependency>`
      ```bash
      docker build -t my_image .
      docker run my_image
      ```
    - **Considerations**: Check logs, update Dockerfile, test after rebuild.

34. **Reduce Docker image size**
    - **Solution**:
      ```dockerfile
      FROM alpine:latest
      RUN apt-get update && apt-get install -y <package> && apt-get clean && rm -rf /var/lib/apt/lists/*
      FROM node:16 AS builder
      WORKDIR /app
      COPY package.json .
      RUN npm install
      FROM node:16-slim
      COPY --from=builder /app /app
      RUN rm -rf /app/unneeded_files
      ```
    - **Considerations**: Use lightweight images, clean up after installs, minimize layers.

35. **Pulling outdated image from registry**
    - **Solution**:
      ```bash
      docker pull my_image:latest
      docker image rm my_image:latest
      docker images my_image
      docker run my_image:latest
      ```
    - **Considerations**: Avoid `latest` in production, clear cache, verify registry credentials.

36. **Build fails due to network timeout**
    - **Solution**:
      ```bash
      ping archive.ubuntu.com
      ```
      - Use mirror: `RUN echo "deb http://mirror.example.com/ubuntu focal main" > /etc/apt/sources.list`
      ```bash
      docker build --build-arg APT_TIMEOUT=300 -t my_image .
      docker build --no-cache -t my_image .
      ```
    - **Considerations**: Verify network, use reliable mirrors, cache packages locally.

37. **Scan for image vulnerabilities**
    - **Solution**:
      ```bash
      trivy image my_image:latest
      ```
      - Update: `RUN apt-get update && apt-get upgrade -y`
      ```bash
      docker build -t my_image:latest .
      trivy image my_image:latest
      ```
    - **Considerations**: Use scanners (Trivy/Clair), keep images updated, retest after updates.

38. **Base image unavailable in registry**
    - **Solution**:
      ```bash
      docker images
      docker tag my_image:backup my_image:latest
      docker search <image_name>
      ```
      - Update: `FROM new_image:latest`
      ```bash
      docker build -t my_image .
      ```
    - **Considerations**: Maintain local backups, use specific tags, test alternatives.

39. **Create multi-architecture image**
    - **Solution**:
      ```bash
      docker buildx create --use
      docker buildx build --platform linux/amd64,linux/arm64 -t my_image:multiarch --push .
      docker buildx imagetools inspect my_image:multiarch
      ```
    - **Considerations**: Ensure buildx installed, specify architectures, push to registry.

40. **Inconsistent Dockerfile builds**
    - **Solution**:
      ```bash
      cat Dockerfile
      ```
      - Pin versions: `FROM node:16.13.2; RUN npm install express@4.17.1`
      ```bash
      docker build --no-cache -t my_image .
      docker run -it <builder_image> bash
      ```
    - **Considerations**: Pin package versions, avoid variable outputs, standardize build environment.

---

## 5. Linux System Administration

41. **Docker daemon fails: "cannot open /var/run/docker.pid"**
    - **Solution**:
      ```bash
      ls -l /var/run/docker.pid
      rm -f /var/run/docker.pid
      systemctl status docker
      systemctl start docker
      journalctl -u docker -b
      ```
    - **Considerations**: Check socket conflicts, permissions, disk space.

42. **Docker service high memory usage**
    - **Solution**:
      ```bash
      top -p $(pidof dockerd)
      docker stats --no-stream
      docker update --memory 512m <container_id>
      ```
    - **Considerations**: Monitor container leaks, adjust daemon config, enable swap.

43. **Server running out of inodes**
    - **Solution**:
      ```bash
      df -i
      find / -xdev -printf '%h\n' | sort | uniq -c | sort -k 1 -nr | head
      docker system prune -a --volumes
      du -sh /var/lib/docker
      ```
    - **Considerations**: Clean unused Docker objects, check storage driver, monitor `/var/lib/docker`.

44. **High I/O wait times on Docker host**
    - **Solution**:
      ```bash
      iostat -x 1
      iotop -o
      docker stats --no-stream
      docker ps -q | xargs -I {} sh -c 'echo Container: {}; cat /sys/fs/cgroup/blkio/docker/{}/blkio.throttle.io_service_bytes'
      ```
    - **Considerations**: Identify high I/O processes, use `ionice`, evaluate storage driver.

45. **Port conflict prevents container start**
    - **Solution**:
      ```bash
      netstat -tulnp | grep :80
      lsof -i :80
      kill -9 <pid>
      docker run -d -p 80:80 <image_name>
      ```
    - **Considerations**: Check container conflicts, reassign ports if needed.

46. **Docker daemon not responding**
    - **Solution**:
      ```bash
      systemctl status docker
      systemctl reload docker
      systemctl restart docker
      docker ps
      ```
    - **Considerations**: Reload preserves containers, check logs (`journalctl -u docker`).

47. **High load averages on Docker host**
    - **Solution**:
      ```bash
      uptime
      top -p $(pidof dockerd)
      docker stats --no-stream
      ps aux --sort=-%cpu | head
      iostat -x 1
      free -m
      ```
    - **Considerations**: Correlate load with Docker, monitor resources, limit containers.

48. **Root filesystem full on Docker host**
    - **Solution**:
      ```bash
      df -h /
      du -ah / | sort -rh | head -n 10
      docker system prune -a --volumes
      docker image prune -f
      truncate -s 0 /var/log/*.log
      ```
    - **Considerations**: Avoid deleting critical data, use `logrotate`, check container logs.

49. **Container fails due to host permissions**
    - **Solution**:
      ```bash
      docker logs <container_id>
      ls -l /var/run/docker.sock
      usermod -aG docker <username>
      getenforce
      aa-status
      docker run --user $(id -u):$(id -g) <image_name>
      ```
    - **Considerations**: Verify socket access, check SELinux/AppArmor, avoid root.

50. **Kernel panic on Docker host**
    - **Solution**:
      ```bash
      cat /var/log/messages | grep -i panic
      journalctl -b -1
      ls -l /var/crash
      crash /var/crash/*/vmcore /usr/lib/debug/lib/modules/$(uname -r)/vmlinux
      ```
    - **Considerations**: Check storage driver/networking, verify kernel updates, enable `kdump`.

---

## 6. Docker Security

51. **Run container as non-root user**
    - **Solution**:
      ```dockerfile
      FROM node:16
      RUN useradd -m appuser
      USER appuser
      COPY . /app
      WORKDIR /app
      CMD ["node", "app.js"]
      ```
      ```bash
      docker run --user appuser <image_name>
      docker build -t <image_name> .
      ```
    - **Considerations**: Verify app supports non-root, check permissions.

52. **Investigate compromised container**
    - **Solution**:
      ```bash
      docker top <container_id>
      docker logs <container_id>
      docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <image_name>
      docker pause <container_id>
      docker rm -f <container_id>
      ```
    - **Considerations**: Quarantine container, check network, rebuild from trusted source.

53. **Remove sensitive credentials from image**
    - **Solution**:
      ```bash
      docker history <image_name>
      docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <image_name>
      ```
      - Rebuild without credentials:
        ```dockerfile
        FROM node:16
        COPY . /app
        WORKDIR /app
        CMD ["node", "app.js"]
        ```
    - **Considerations**: Use secrets/environment variables, squash layers.

54. **Container bypasses security policies**
    - **Solution**:
      ```bash
      docker inspect <container_id> | grep CapAdd
      docker inspect <container_id> | grep Privileged
      docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE <image_name>
      docker run --security-opt apparmor=docker-default <image_name>
      ```
    - **Considerations**: Avoid privileged mode, apply AppArmor/SELinux, audit configs.

55. **Restrict container filesystem access**
    - **Solution**:
      ```bash
      docker run --read-only <image_name>
      docker run -v /host/path:/container/path:ro <image_name>
      docker run --tmpfs /tmp <image_name>
      ```
    - **Considerations**: Avoid sensitive mounts, use read-only, combine with user namespaces.

56. **Update vulnerable library in container**
    - **Solution**:
      ```bash
      docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <image_name>
      ```
      - Update: `FROM node:16; RUN apt-get update && apt-get upgrade -y`
      ```bash
      docker build -t <image_name>:latest .
      docker push <image_name>:latest
      ```
    - **Considerations**: Update base images, automate scans, test compatibility.

57. **Block unauthorized container network requests**
    - **Solution**:
      ```bash
      docker network inspect <network_name>
      tcpdump -i docker0
      iptables -A OUTPUT -s <container_ip> -j DROP
      docker run --network none <image_name>
      ```
    - **Considerations**: Use network namespaces, firewall rules, audit ports.

58. **Implement Docker role-based access control**
    - **Solution**:
      ```bash
      useradd -m dockeruser
      groupadd docker
      usermod -aG docker dockeruser
      chown root:docker /var/run/docker.sock
      chmod 660 /var/run/docker.sock
      docker context create restricted --docker host=unix:///var/run/docker.sock
      ```
    - **Considerations**: Limit `docker` group, use TLS, audit permissions.

59. **Reduce container capabilities**
    - **Solution**:
      ```bash
      docker inspect <container_id> | grep CapAdd
      docker run --cap-drop=ALL --cap-add=CHOWN --cap-add=SETUID <image_name>
      docker exec <container_id> capsh --print
      ```
    - **Considerations**: Drop unnecessary capabilities, test functionality.

60. **Verify registry for malicious images**
    - **Solution**:
      ```bash
      docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <registry>/<image_name>
      docker trust inspect <registry>/<image_name>
      docker pull --disable-content-trust=false <registry>/<image_name>
      docker login <registry>
      ```
    - **Considerations**: Use trusted registries, enable Docker Content Trust, scan regularly.

---

## 7. Docker Orchestration (Docker Compose/Swarm)

61. **Docker Compose "service not found" error**
    - Check `docker-compose.yml` for service definition, validate syntax (`docker-compose config`), restart (`docker-compose up -d`).
    - **Considerations**: Verify service names, file paths, and Docker Compose version.

62. **Swarm service stuck in "pending"**
    - Check node resources (`docker node ls`), logs (`docker service logs <service>`), and constraints. Scale or adjust (`docker service update`).
    - **Considerations**: Ensure sufficient nodes, check placement constraints.

63. **Compose service communication failure**
    - Verify network (`docker network inspect <network>`), test connectivity (`docker exec <container> ping <service>`), ensure service names resolve.
    - **Considerations**: Check DNS, firewall, and network configuration.

64. **Swarm service not scaling**
    - Check resources (`docker node ls`), update replicas (`docker service scale <service>=3`), review logs (`docker service logs <service>`).
    - **Considerations**: Ensure node capacity, check constraints.

65. **Compose ignores environment variables**
    - Verify `docker-compose.yml` (`env_file` or `environment`), check container (`docker inspect <container> | grep Env`), restart (`docker-compose up -d`).
    - **Considerations**: Validate variable syntax, avoid overrides.

66. **Swarm node marked "Down"**
    - Check status (`docker node ls`), logs (`journalctl -u docker`), restart node (`docker node update --availability active <node>`).
    - **Considerations**: Verify network, Docker daemon, and node health.

67. **Rolling update for Swarm service**
    - Update service: `docker service update --image <image>:latest <service> --update-parallelism 1 --update-delay 10s`
    - **Considerations**: Test updates, ensure health checks, monitor (`docker service ps <service>`).

68. **Compose crashes due to resource limits**
    - Check limits (`docker inspect <container>`), adjust `docker-compose.yml` (`mem_limit: 512m`, `cpus: 0.5`), restart (`docker-compose up -d`).
    - **Considerations**: Monitor resource usage, balance limits.

69. **Swarm service not load balancing**
    - Check routing mesh (`docker network inspect ingress`), verify replicas (`docker service ps <service>`), test endpoints (`curl <node_ip>`).
    - **Considerations**: Ensure correct ports, check load balancer config.

70. **Compose file startup errors**
    - Validate syntax (`docker-compose config`), check logs (`docker-compose logs`), fix errors, restart (`docker-compose up -d`).
    - **Considerations**: Verify service dependencies, image availability.

---

## 8. Performance and Monitoring

71. **High latency in Dockerized app**
    - **Solution**:
      ```bash
      docker stats
      top -p $(pidof dockerd)
      iostat -x 1
      netstat -i
      ```
    - Check CPU, memory, disk, or network bottlenecks.
    - **Considerations**: Correlate container and host metrics.

72. **Periodic memory spikes**
    - **Solution**:
      ```bash
      docker stats <container_name>
      docker inspect <container_name> | grep Memory
      smem -k -p -c pss,uss
      ```
    - Monitor spikes, check limits, analyze processes.
    - **Considerations**: Verify application activity, check for leaks.

73. **Analyze high load with Linux tools**
    - **Solution**:
      ```bash
      top
      htop
      sar -u 1 5
      sar -r 1 5
      ```
    - Identify high-CPU/memory processes, check trends.
    - **Considerations**: Look for Docker processes, monitor historical data.

74. **Slow containerized app**
    - **Solution**:
      ```bash
      docker stats <container_name>
      docker top <container_name>
      iostat -x 1
      netstat -tunap
      ```
    - Check resources, processes, disk, and network.
    - **Considerations**: Correlate with logs, identify contention.

75. **Set up monitoring**
    - **Solution**:
      ```bash
      docker run -d --name prometheus -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
      docker run -d --name grafana -p 3000:3000 grafana/grafana
      ```
    - Monitor CPU, memory, I/O, network with Prometheus/Grafana.
    - **Considerations**: Configure Prometheus scraping, set alerts.

76. **Excessive container disk I/O**
    - **Solution**:
      ```bash
      iostat -x 1
      iotop -o -P
      docker inspect <container_name> | grep Mounts
      ```
    - Identify high I/O processes, check mounts.
    - **Considerations**: Optimize app I/O, check host issues.

77. **Network bottlenecks on Docker host**
    - **Solution**:
      ```bash
      iftop -i eth0
      nload eth0
      docker network inspect bridge
      ```
    - Monitor traffic, check container network config.
    - **Considerations**: Identify high-traffic containers, check policies.

78. **Benchmark container performance**
    - **Solution**:
      ```bash
      docker run -it <container_name> stress --cpu 2 --io 1 --vm 1 --vm-bytes 128M --timeout 60s
      docker stats <container_name>
      perf stat -p $(docker top <container_name> | awk 'NR>1 {print $2}')
      ```
    - Simulate load, monitor metrics.
    - **Considerations**: Use realistic workloads, compare baselines.

79. **Capture intermittent container crashes**
    - **Solution**:
      ```bash
      docker logs <container_name> --follow
      docker events --filter 'container=<container_name>'
      docker run -d --name promtail -v /var/log:/var/log loki/promtail -config.file=/etc/promtail/promtail.yml
      ```
    - Stream logs, monitor events, use Promtail/Loki.
    - **Considerations**: Correlate crashes with resources, ensure log capture.

80. **Profile container CPU usage**
    - **Solution**:
      ```bash
      docker stats <container_name>
      docker top <container_name>
      perf stat -p $(docker top <container_name> | awk 'NR>1 {print $2}')
      docker update --cpus="0.5" <container_name>
      ```
    - Monitor and limit CPU usage.
    - **Considerations**: Identify intensive processes, balance limits.

---

## 9. Logging and Debugging

81. **No logs from containerized app**
    - **Solution**:
      ```bash
      docker inspect <container_name> | grep LogConfig
      docker run --log-driver=json-file --log-opt max-size=10m <container_name>
      docker logs <container_name>
      ```
    - Configure `json-file` driver, verify logs.
    - **Considerations**: Ensure app writes to stdout/stderr, check driver compatibility.

82. **Collect logs from multiple containers**
    - **Solution**:
      ```bash
      docker run -d --name loki -p 3100:3100 grafana/loki
      docker run -d --name promtail -v /var/lib/docker/containers:/var/lib/docker/containers loki/promtail -config.file=/etc/promtail/promtail.yml
      ```
    - Use Loki/Promtail for aggregation, query via Grafana.
    - **Considerations**: Ensure Promtail access, configure retention.

83. **Increase log verbosity for crashes**
    - **Solution**:
      ```bash
      docker inspect <container_name> | grep LogConfig
      docker run --log-driver=json-file --log-opt max-size=50m <container_name> --verbose
      docker logs <container_name>
      ```
    - Enable verbose mode, increase log size.
    - **Considerations**: Verify app supports verbose, monitor storage.

84. **Debug container process with `strace`/`gdb`**
    - **Solution**:
      ```bash
      docker exec -it <container_name> apt-get update && apt-get install -y strace gdb
      docker exec -it <container_name> strace -p $(pgrep process_name)
      docker exec -it <container_name> gdb --pid=$(pgrep process_name)
      ```
    - Install and use debugging tools.
    - **Considerations**: Ensure tools available, avoid impacting production.

85. **Redirect container logs to file**
    - **Solution**:
      ```bash
      docker run --log-driver=json-file --log-opt max-size=10m -v /host/path:/logs <container_name>
      docker exec <container_name> sh -c 'echo "log output" > /logs/app.log'
      ```
    - Use volume for log files.
    - **Considerations**: Check permissions, monitor log growth.

86. **Confirm container memory leak**
    - **Solution**:
      ```bash
      docker exec -it <container_name> apt-get update && apt-get install -y valgrind
      docker exec -it <container_name> valgrind --leak-check=full --log-file=/tmp/valgrind.log process_name
      ```
    - Use `valgrind` to detect leaks.
    - **Considerations**: Ensure debug symbols, account for overhead.

87. **Extract context from cryptic errors**
    - **Solution**:
      ```bash
      docker logs <container_name> --tail 100
      docker inspect <container_name> | grep Env
      docker exec <container_name> cat /path/to/config
      ```
    - Review logs, environment, and configs.
    - **Considerations**: Correlate errors with settings, enable debug mode.

88. **Centralize logs from multiple hosts**
    - **Solution**:
      ```bash
      docker run -d --name loki -p 3100:3100 grafana/loki
      docker run -d --name promtail -v /var/lib/docker/containers:/var/lib/docker/containers loki/promtail -config.file=/etc/promtail/promtail.yml
      ```
    - Use Loki/Promtail, query via Grafana.
    - **Considerations**: Ensure connectivity, secure with TLS.

89. **Enable debugging for silent failures**
    - **Solution**:
      ```bash
      docker run --log-driver=json-file --log-opt max-size=10m <container_name> --debug
      docker events --filter 'container=<container_name>'
      docker logs <container_name>
      ```
    - Enable debug, monitor events/logs.
    - **Considerations**: Verify debug support, check config for suppressed logs.

90. **Manage excessive journald logs**
    - **Solution**:
      ```bash
      journalctl --disk-usage
      journalctl --vacuum-size=500M
      sudo vi /etc/systemd/journald.conf
      # Set: SystemMaxUse=500M
      sudo systemctl restart systemd-journald
      ```
    - Reduce log size, set limits.
    - **Considerations**: Balance retention, monitor post-changes.

---

## 10. Disaster Recovery and High Availability

91. **Auto-restart containers after host crash**
    - **Solution**:
      ```bash
      sudo systemctl enable docker
      docker run --restart=always -d <image_name>
      docker update --restart=always <container_id>
      ```
    - **Considerations**: Use appropriate restart policy, verify resources, check logs.

92. **Restore critical containerized app**
    - **Solution**:
      ```bash
      docker-compose -f docker-compose.yml up -d
      docker run -d --name <container_name> -p <port>:<port> -v <volume> <image_name>
      docker ps -a
      ```
    - **Considerations**: Store configs, ensure image/volume availability, test restoration.

93. **Backup Dockerized database**
    - **Solution**:
      ```bash
      docker exec <mysql_container> mysqldump -u root -p<password> --all-databases > backup.sql
      scp backup.sql user@remote_host:/backups/
      echo "0 2 * * * docker exec <mysql_container> mysqldump -u root -p<password> --all-databases > /backups/backup_$(date +\%F).sql" | crontab -
      ```
    - **Considerations**: Use app-specific tools, encrypt backups, test restoration.

94. **Recover Swarm cluster after manager loss**
    - **Solution**:
      ```bash
      docker node ls
      docker node promote <worker_node_id>
      docker swarm init --force-new-cluster --advertise-addr <manager_ip>:2377
      docker swarm join --token <worker_token> <manager_ip>:2377
      ```
    - **Considerations**: Maintain odd-numbered managers, back up Swarm state, verify health.

95. **Restore corrupted volume**
    - **Solution**:
      ```bash
      docker stop <container_id>
      tar -xzf /backups/volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>
      docker start <container_id>
      docker exec <container_id> ls /data
      ```
    - **Considerations**: Back up regularly, verify integrity, test app post-restore.

96. **Recover Docker host from malware**
    - **Solution**:
      ```bash
      sudo ip link set eth0 down
      docker stop $(docker ps -q)
      sudo apt install clamav
      clamscan -r / --remove
      sudo apt update && sudo apt upgrade -y
      sudo systemctl restart docker
      docker start $(docker ps -a -q)
      ```
    - **Considerations**: Block malicious traffic, scan images, restore from clean backups.

97. **Ensure high availability**
    - **Solution**:
      ```bash
      docker swarm init
      docker service create --name app --replicas 3 -p 80:80 <image_name>
      HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost/ || exit 1
      docker service update --health-cmd="curl -f http://localhost/ || exit 1" --health-interval=30s app
      ```
    - **Considerations**: Use multiple nodes, load balancer, monitor health.

98. **Recover from filesystem corruption**
    - **Solution**:
      ```bash
      sudo fsck /dev/sdX
      sudo apt remove docker.io
      sudo apt install docker.io
      docker-compose -f docker-compose.yml up -d
      tar -xzf /backups/volume_backup.tar.gz -C /var/lib/docker/volumes/
      ```
    - **Considerations**: Back up `/var/lib/docker`, verify filesystem, test containers.

99. **Implement failover for frequent outages**
    - **Solution**:
      ```bash
      docker swarm init
      docker service create --name app --replicas 3 -p 80:80 <image_name>
      ```
      - NGINX config:
        ```nginx
        upstream app {
            server <node1_ip>:80;
            server <node2_ip>:80;
            server <node3_ip>:80;
        }
        server { listen 80; location / { proxy_pass http://app; } }
        ```
      ```bash
      sudo systemctl reload nginx
      ```
    - **Considerations**: Spread replicas, use health checks, test failover.

100. **Simulate Docker host failure**
     - **Solution**:
       ```bash
       sudo systemctl stop docker
       docker node update --availability drain <node_id>
       docker service ps <service_name>
       sudo systemctl start docker
       docker node update --availability active <node_id>
       ```
     - **Considerations**: Ensure replicas, test backups, monitor availability, document steps.

---
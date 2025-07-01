# Docker & Linux On-Call Troubleshooting and Scenario-Based Interview Questions

---

## 1. Docker Container Management
These answers focus on creating, managing, and troubleshooting Docker containers.

1. **A container fails to start with the error: "Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused 'exec: "/bin/bash": stat /bin/bash: no such file or directory'." How would you troubleshoot this?**

   **Solution**: The error indicates the container's image lacks `/bin/bash` or the entrypoint is incorrectly set.
   - Check the Dockerfile or image entrypoint:
     ```bash
     docker inspect <image_name> | grep -i entrypoint
     ```
   - Verify if `/bin/bash` exists in the image:
     ```bash
     docker run -it <image_name> /bin/sh -c "ls /bin/bash"
     ```
   - If missing, use `/bin/sh` or update the Dockerfile to include bash:
     ```dockerfile
     FROM ubuntu:20.04
     RUN apt-get update && apt-get install -y bash
     ```
   - Alternatively, modify the entrypoint to use an available shell:
     ```bash
     docker run -it --entrypoint /bin/sh <image_name>
     ```

   **Key Considerations**:
   - Ensure the base image includes the required shell.
   - Minimal images (e.g., alpine) may not have bash; use sh instead.
   - Validate the entrypoint and command in the Dockerfile or runtime.

2. **You deploy a container, but it exits immediately with code 137. What does this indicate, and how would you resolve it?**

   **Solution**: Exit code 137 means the container was killed due to excessive memory usage (OOM killer).
   - Check container logs:
     ```bash
     docker logs <container_id>
     ```
   - Inspect memory limits:
     ```bash
     docker inspect <container_id> | grep -i memory
     ```
   - Increase memory limit:
     ```bash
     docker run --memory="512m" <image_name>
     ```
   - Optimize the application or image to reduce memory usage.

   **Key Considerations**:
   - Monitor host memory with `free -m` to ensure sufficient resources.
   - Check for memory leaks in the application.
   - Use `--memory-swap` to control swap usage if needed.

3. **A developer reports that a container is stuck in a "Created" state and never starts. What steps would you take to diagnose and fix this?**

   **Solution**:
   - Check container status:
     ```bash
     docker ps -a
     ```
   - Inspect container for errors:
     ```bash
     docker inspect <container_id> | jq '.State.Error'
     ```
   - Check Docker daemon logs:
     ```bash
     journalctl -u docker.service
     ```
   - Start the container manually:
     ```bash
     docker start <container_id>
     ```
   - If it fails, recreate the container:
     ```bash
     docker rm <container_id> && docker run <image_name>
     ```

   **Key Considerations**:
   - Ensure Docker daemon is running (`systemctl status docker`).
   - Check for resource constraints or misconfigured Docker options.
   - Verify the image is compatible with the host architecture.

4. **A container is running, but the application inside is not responding. How would you debug the issue without stopping the container?**

   **Solution**:
   - Access the container:
     ```bash
     docker exec -it <container_id> /bin/sh
     ```
   - Check running processes:
     ```bash
     ps aux
     ```
   - Inspect application logs:
     ```bash
     docker logs <container_id>
     ```
   - Test application endpoints (e.g., curl from inside):
     ```bash
     curl http://localhost:<port>
     ```
   - Check resource usage:
     ```bash
     docker stats <container_id>
     ```

   **Key Considerations**:
   - Ensure the application is running and listening on the correct port.
   - Check for application-specific errors in logs or configuration.
   - Avoid stopping the container to preserve the current state for debugging.

5. **You notice that a container is consuming excessive CPU. How would you identify the cause and mitigate it?**

   **Solution**:
   - Monitor container CPU usage:
     ```bash
     docker stats <container_id>
     ```
   - Identify processes inside the container:
     ```bash
     docker exec <container_id> top
     ```
   - Limit CPU usage:
     ```bash
     docker update --cpus="0.5" <container_id>
     ```
   - Investigate application behavior (e.g., check for infinite loops).

   **Key Considerations**:
   - Use `--cpu-quota` or `--cpus` during `docker run` for proactive limits.
   - Check for misconfigured workloads or resource-intensive tasks.
   - Monitor host CPU with `htop` to rule out other causes.

6. **A container is not accessible, and you suspect it’s not running. How would you verify its status and restart it if necessary?**

   **Solution**:
   - Check container status:
     ```bash
     docker ps -a | grep <container_name>
     ```
   - If not running, restart it:
     ```bash
     docker start <container_id>
     ```
   - If it fails, check logs for errors:
     ```bash
     docker logs <container_id>
     ```
   - Recreate if necessary:
     ```bash
     docker rm <container_id> && docker run <image_name>
     ```

   **Key Considerations**:
   - Verify the container’s configuration (ports, volumes).
   - Ensure the Docker daemon is running.
   - Check for host resource issues (e.g., `df -h`, `free -m`).

7. **A container was deleted accidentally, but you have its image. How would you recreate it with the same configuration?**

   **Solution**:
   - Check the original run command (if logged):
     ```bash
     docker inspect <image_name> | grep -i cmd
     ```
   - Recreate with the same parameters (e.g., ports, volumes):
     ```bash
     docker run -d --name <container_name> -p <host_port>:<container_port> -v <volume_path> <image_name>
     ```
   - If using Docker Compose, re-run:
     ```bash
     docker-compose up -d
     ```

   **Key Considerations**:
   - Store container configurations in Docker Compose or scripts for reproducibility.
   - Verify volume data persists if used.
   - Check for environment variables or custom entrypoints.

8. **You’re tasked with scaling a Dockerized application to handle more traffic. How would you approach this using Docker?**

   **Solution**:
   - Use Docker Compose to scale services:
     ```bash
     docker-compose up -d --scale <service_name>=3
     ```
   - Or use Docker Swarm:
     ```bash
     docker swarm init
     docker service create --replicas 3 --name <service_name> <image_name>
     ```
   - Set up a load balancer (e.g., nginx):
     ```bash
     docker run -d -p 80:80 nginx
     ```
     Configure nginx.conf to balance traffic:
     ```nginx
     upstream app {
         server container1:8080;
         server container2:8080;
         server container3:8080;
     }
     server {
         listen 80;
         location / {
             proxy_pass http://app;
         }
     }
     ```

   **Key Considerations**:
   - Ensure the application is stateless or uses shared storage.
   - Monitor resource usage with `docker stats`.
   - Use an external load balancer for production (e.g., AWS ELB).

9. **A container fails with "Error: cannot allocate memory." What could be the cause, and how would you resolve it?**

   **Solution**:
   - Check host memory:
     ```bash
     free -m
     ```
   - Inspect container memory limits:
     ```bash
     docker inspect <container_id> | grep -i memory
     ```
   - Increase memory limit:
     ```bash
     docker run --memory="1g" <image_name>
     ```
   - Optimize application memory usage or reduce container load.

   **Key Considerations**:
   - Check for memory leaks in the application.
   - Ensure host has sufficient memory or adjust swap settings.
   - Use `docker system prune` to free up resources.

10. **A container is running an outdated version of an application. How would you update it to the latest version without downtime?**

   **Solution**:
   - Pull the latest image:
     ```bash
     docker pull <image_name>:latest
     ```
   - Use Docker Compose for zero-downtime deployment:
     ```bash
     docker-compose up -d
     ```
   - Or use Docker Swarm for rolling updates:
     ```bash
     docker service update --image <image_name>:latest <service_name>
     ```
   - Verify the new version:
     ```bash
     docker exec <container_id> <app_version_command>
     ```

   **Key Considerations**:
   - Test the new image in a staging environment first.
   - Ensure backward compatibility for data and configurations.
   - Use health checks to confirm the new version is stable.

---

## 2. Docker Networking
These answers focus on troubleshooting and configuring Docker networking.

11. **A container cannot connect to an external database. How would you troubleshoot the network connectivity issue?**

   **Solution**:
   - Verify container network:
     ```bash
     docker inspect <container_id> | grep -i network
     ```
   - Test connectivity from the container:
     ```bash
     docker exec <container_id> ping <db_host>
     ```
   - Check DNS resolution:
     ```bash
     docker exec <container_id> nslookup <db_host>
     ```
   - Ensure firewall allows traffic:
     ```bash
     sudo iptables -L -n
     ```
   - Add database host to container’s network or update firewall rules:
     ```bash
     sudo iptables -A INPUT -p tcp --dport <db_port> -j ACCEPT
     ```

   **Key Considerations**:
   - Verify database host and port are correct.
   - Check if the container uses the correct network mode (e.g., bridge, host).
   - Ensure security groups or firewall rules allow outbound traffic.

12. **Two containers in the same Docker network cannot communicate with each other. What could be the issue, and how would you fix it?**

   **Solution**:
   - Verify both containers are in the same network:
     ```bash
     docker network ls
     docker inspect <container_id> | grep -i network
     ```
   - Test connectivity:
     ```bash
     docker exec <container1_id> ping <container2_name>
     ```
   - If using Docker Compose, ensure network is defined:
     ```yaml
     version: '3'
     services:
       app1:
         image: <image1>
         networks:
           - mynet
       app2:
         image: <image2>
         networks:
           - mynet
     networks:
       mynet:
         driver: bridge
     ```
   - Recreate the network if needed:
     ```bash
     docker network create mynet
     docker network connect mynet <container_id>
     ```

   **Key Considerations**:
   - Use container names for DNS resolution in the same network.
   - Check for network policies or firewall rules blocking traffic.
   - Restart containers after network changes.

13. **A containerized application is not accessible from the host machine via its mapped port. How would you diagnose this?**

   **Solution**:
   - Verify port mapping:
     ```bash
     docker inspect <container_id> | grep -i port
     ```
   - Check if the application is listening:
     ```bash
     docker exec <container	context_id> netstat -tuln
     ```
   - Test from host:
     ```bash
     curl http://localhost:<host_port>
     ```
   - Check host firewall:
     ```bash
     sudo iptables -L -n
     ```
   - Open port if blocked:
     ```bash
     sudo iptables -A INPUT -p tcp --dport <host_port> -j ACCEPT
     ```

   **Key Considerations**:
   - Ensure the container’s port is correctly mapped (`-p <host_port>:<container_port>`).
   - Verify the application binds to `0.0.0.0`, not `127.0.0.1`.
   - Check for conflicting services on the host.

14. **You see the error: "Bind for 0.0.0.0:80 failed: port is already allocated." How would you resolve this conflict?**

   **Solution**:
   - Identify the process using the port:
     ```bash
     sudo netstat -tulnp | grep :80
     ```
   - Kill the conflicting process:
     ```bash
     sudo kill <pid>
     ```
   - Or run the container on a different port:
     ```bash
     docker run -p 8080:80 <image_name>
     ```

   **Key Considerations**:
   - Check if another container or service is using the port.
   - Use `docker ps` to identify running containers with port conflicts.
   - Avoid running multiple services on the same host port.

15. **A container is unable to resolve DNS names. What steps would you take to troubleshoot and fix this?**

   **Solution**:
   - Test DNS resolution:
     ```bash
     docker exec <container_id> nslookup google.com
     ```
   - Check container’s DNS settings:
     ```bash
     docker inspect <container_id> | grep -i dns
     ```
   - Set custom DNS (e.g., Google DNS):
     ```bash
     docker run --dns 8.8.8.8 <image_name>
     ```
   - Verify host DNS configuration:
     ```bash
     cat /etc/resolv.conf
     ```

   **Key Considerations**:
   - Ensure the host’s DNS is configured correctly.
   - Check for network policies blocking DNS traffic (port 53).
   - Restart the container after DNS changes.

16. **A Docker container is dropping packets intermittently. How would you investigate this issue?**

   **Solution**:
   - Check container network stats:
     ```bash
     docker inspect <container_id> | grep -i network
     ```
   - Use tcpdump to capture packets:
     ```bash
     docker exec <container_id> tcpdump -i eth0
     ```
   - Monitor host network:
     ```bash
     sudo tcpdump -i docker0
     ```
   - Check for resource contention:
     ```bash
     docker stats <container_id>
     ```

   **Key Considerations**:
   - Look for network congestion or misconfigured MTU settings.
   - Check Docker network driver (e.g., bridge, overlay).
   - Monitor host network interfaces for errors (`ifconfig`).

17. **You need to restrict network traffic between containers in a Docker network. How would you implement this?**

   **Solution**:
   - Create a custom network with no default connectivity:
     ```bash
     docker network create --driver bridge --opt com.docker.network.bridge.enable_icc=false mynet
     ```
   - Connect containers to the network:
     ```bash
     docker network connect mynet <container_id>
     ```
   - Use iptables to allow specific traffic:
     ```bash
     sudo iptables -A FORWARD -s <container1_ip> -d <container2_ip> -p tcp --dport <port> -j ACCEPT
     ```

   **Key Considerations**:
   - Test connectivity after applying rules.
   - Use Docker Compose networks for easier management.
   - Ensure rules don’t block critical traffic.

18. **A containerized web server is responding with a 502 Bad Gateway error. How would you troubleshoot the networking setup?**

   **Solution**:
   - Check web server logs:
     ```bash
     docker logs <container_id>
     ```
   - Verify backend service is running:
     ```bash
     docker exec <web_container_id> curl http://<backend_service>:<port>
     ```
   - Inspect network configuration:
     ```bash
     docker inspect <container_id> | grep -i network
     ```
   - Ensure proxy configuration (e.g., nginx) points to the correct backend:
     ```nginx
     server {
         listen 80;
         location / {
             proxy_pass http://<backend_service>:<port>;
         }
     }
     ```

   **Key Considerations**:
   - Verify backend service health and port.
   - Check for network misconfigurations or DNS issues.
   - Ensure load balancer or proxy is correctly configured.

19. **You notice that a container’s network performance is slow. What tools and steps would you use to diagnose this?**

   **Solution**:
   - Monitor container network stats:
     ```bash
     docker stats <container_id>
     ```
   - Use iperf to test bandwidth:
     ```bash
     docker run -it --rm <image_name> iperf -c <target_ip>
     ```
   - Check host network performance:
     ```bash
     sudo iftop -i docker0
     ```
   - Inspect MTU settings:
     ```bash
     docker network inspect <network_name> | grep -i mtu
     ```

   **Key Considerations**:
   - Compare performance with other containers or hosts.
   - Check for network congestion or QoS policies.
   - Ensure the correct network driver is used.

20. **A container is unable to access an external API due to a firewall rule. How would you identify and resolve this?**

   **Solution**:
   - Check firewall rules on the host:
     ```bash
     sudo iptables -L -n -v
     ```
   - Test connectivity from the container:
     ```bash
     docker exec <container_id> curl <api_url>
     ```
   - Allow outbound traffic to the API:
     ```bash
     sudo iptables -A OUTPUT -p tcp --dport <api_port> -j ACCEPT
     ```
   - Verify host network settings:
     ```bash
     ip route
     ```

   **Key Considerations**:
   - Ensure the firewall rule targets the correct port and protocol.
   - Check cloud provider security groups if applicable.
   - Test connectivity after rule changes.

---

## 3. Docker Storage and Volumes
These questions focus on managing and troubleshooting Docker storage and volumes.

21. **A container is unable to write to a mounted volume. How would you troubleshoot the issue?**  
   - Check volume permissions:  
     ```bash
     ls -ld /path/to/volume
     chmod -R 775 /path/to/volume
     chown -R user:group /path/to/volume
     ```
   - Verify container user ID matches host volume permissions:  
     ```bash
     docker inspect <container_id> | grep -i uid
     ```
   - Check if volume is mounted correctly:  
     ```bash
     docker inspect <container_id> | jq '.Mounts'
     ```
   - Ensure SELinux/AppArmor isn’t blocking:  
     ```bash
     setenforce 0  # Temporarily disable SELinux
     ```
   - Restart container with updated permissions:  
     ```bash
     docker restart <container_id>
     ```
   - **Key Considerations**: Verify user/group permissions align between host and container. Check for SELinux/AppArmor restrictions. Ensure volume path exists and is accessible.

22. **You notice that a container’s volume is filling up disk space on the host. How would you clean it up safely?**  
   - Identify volume usage:  
     ```bash
     du -sh /var/lib/docker/volumes/*
     ```
   - Stop container to avoid data corruption:  
     ```bash
     docker stop <container_id>
     ```
   - Remove unnecessary files inside volume:  
     ```bash
     docker run --rm -v <volume_name>:/data busybox sh -c "rm -rf /data/unneeded_files"
     ```
   - Prune unused volumes:  
     ```bash
     docker volume prune
     ```
   - **Key Considerations**: Always stop containers before modifying volumes to prevent data loss. Verify which files are safe to delete. Use pruning cautiously to avoid removing active volumes.

23. **A volume used by a container is corrupted. How would you recover the data or mitigate the issue?**  
   - Check volume status:  
     ```bash
     docker volume inspect <volume_name>
     ```
   - Mount volume in a recovery container:  
     ```bash
     docker run --rm -v <volume_name>:/data -it busybox sh
     ```
   - Attempt file system repair:  
     ```bash
     fsck /path/to/volume
     ```
   - Restore from backup if available:  
     ```bash
     tar -xzf /backup/volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>/_data
     ```
   - Recreate container with new volume if unrecoverable:  
     ```bash
     docker volume create new_volume
     docker run -v new_volume:/app <image_name>
     ```
   - **Key Considerations**: Always maintain backups for critical volumes. Use recovery containers to inspect data. Test restored data before resuming operations.

24. **A container fails to start with the error: "no such file or directory" for a mounted volume. What could be wrong?**  
   - Verify volume exists:  
     ```bash
     docker volume ls
     ```
   - Check host path for bind mounts:  
     ```bash
     ls -ld /path/to/host/mount
     ```
   - Create missing directory:  
     ```bash
     mkdir -p /path/to/host/mount
     ```
   - Fix volume in container run command:  
     ```bash
     docker run -v /path/to/host/mount:/app <image_name>
     ```
   - Check Dockerfile for incorrect volume declarations:  
     ```bash
     cat Dockerfile | grep VOLUME
     ```
   - **Key Considerations**: Ensure the host path or volume exists before starting the container. Verify path spelling and permissions. Check for typos in Docker commands or Dockerfile.

25. **You need to migrate a Docker volume to a new host. How would you accomplish this?**  
   - Stop container using the volume:  
     ```bash
     docker stop <container_id>
     ```
   - Back up volume data:  
     ```bash
     tar -czf volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>/_data .
     ```
   - Transfer backup to new host:  
     ```bash
     scp volume_backup.tar.gz user@new_host:/tmp
     ```
   - Create volume on new host:  
     ```bash
     docker volume create <volume_name>
     ```
   - Restore data:  
     ```bash
     tar -xzf /tmp/volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>/_data
     ```
   - Start container on new host:  
     ```bash
     docker run -v <volume_name>:/app <image_name>
     ```
   - **Key Considerations**: Ensure compatibility of Docker versions between hosts. Verify file permissions after transfer. Test container functionality post-migration.

26. **A container is using a bind mount, but changes made inside the container are not reflected on the host. Why might this happen, and how would you fix it?**  
   - Check bind mount configuration:  
     ```bash
     docker inspect <container_id> | jq '.Mounts'
     ```
   - Verify host path permissions:  
     ```bash
     ls -ld /path/to/host/mount
     chmod -R 775 /path/to/host/mount
     ```
   - Ensure container user has write access:  
     ```bash
     docker exec <container_id> id
     ```
   - Remount with correct permissions:  
     ```bash
     docker run -v /path/to/host/mount:/app:rw <image_name>
     ```
   - Check for SELinux issues:  
     ```bash
     chcon -R -t container_file_t /path/to/host/mount
     ```
   - **Key Considerations**: Confirm read-write (`rw`) flag in mount. Align container and host user permissions. Check for filesystem restrictions like SELinux.

27. **You notice that unused Docker volumes are consuming significant disk space. How would you identify and remove them?**  
   - List all volumes:  
     ```bash
     docker volume ls
     ```
   - Identify unused volumes:  
     ```bash
     docker volume ls -f dangling=true
     ```
   - Remove unused volumes:  
     ```bash
     docker volume prune
     ```
   - Verify disk space freed:  
     ```bash
     df -h
     ```
   - **Key Considerations**: Ensure no running containers depend on volumes before pruning. Double-check volume usage to avoid accidental data loss.

28. **A containerized database is running out of disk space. How would you expand its storage without downtime?**  
   - Check current disk usage:  
     ```bash
     docker exec <container_id> df -h
     ```
   - Create a new, larger volume:  
     ```bash
     docker volume create new_volume
     ```
   - Copy data to new volume:  
     ```bash
     docker run --rm -v <old_volume>:/old -v new_volume:/new busybox sh -c "cp -a /old/. /new/"
     ```
   - Update container to use new volume:  
     ```bash
     docker stop <container_id>
     docker run -v new_volume:/data <image_name>
     ```
   - **Key Considerations**: Minimize downtime by performing copy operation quickly. Verify data integrity after migration. Ensure sufficient disk space on host.

29. **A container cannot access a mounted NFS volume. How would you troubleshoot this issue?**  
   - Verify NFS server is reachable:  
     ```bash
     ping <nfs_server_ip>
     ```
   - Check NFS mount on host:  
     ```bash
     mount | grep nfs
     ```
   - Ensure NFS client is installed:  
     ```bash
     apt-get install -y nfs-common
     ```
   - Test NFS mount manually:  
     ```bash
     mount -t nfs <nfs_server_ip>:/share /mnt
     ```
   - Check container mount configuration:  
     ```bash
     docker inspect <container_id> | jq '.Mounts'
     ```
   - Restart container with correct NFS mount:  
     ```bash
     docker run -v <nfs_server_ip>:/share:/mnt <image_name>
     ```
   - **Key Considerations**: Confirm NFS server availability and permissions. Check firewall rules and network connectivity. Ensure NFS client tools are installed on the host.

30. **You need to ensure that a container’s data persists across container restarts. How would you configure this?**  
   - Create a named volume:  
     ```bash
     docker volume create persistent_volume
     ```
   - Run container with volume:  
     ```bash
     docker run -v persistent_volume:/app/data <image_name>
     ```
   - Verify volume persistence:  
     ```bash
     docker inspect persistent_volume
     ```
   - **Key Considerations**: Use named volumes instead of bind mounts for portability. Ensure volume is backed up for critical data. Verify volume is mounted correctly in container.

---

## 4. Docker Image Management
These questions focus on building, managing, and troubleshooting Docker images.

31. **A Docker build fails with the error: "COPY failed: stat /var/lib/docker/tmp/xyz: no such file or directory." How would you resolve this?**  
   - Check Dockerfile for incorrect path:  
     ```bash
     cat Dockerfile | grep COPY
     ```
   - Verify file exists in build context:  
     ```bash
     ls -l ./xyz
     ```
   - Fix path in Dockerfile:  
     ```dockerfile
     COPY ./xyz /app/xyz
     ```
   - Rebuild image:  
     ```bash
     docker build -t my_image .
     ```
   - **Key Considerations**: Ensure files referenced in COPY exist in the build context. Check for typos in file paths. Use relative paths correctly.

32. **An image build is taking too long. How would you optimize the Dockerfile to reduce build time?**  
   - Cache dependencies:  
     ```dockerfile
     COPY requirements.txt .
     RUN pip install -r requirements.txt
     COPY . .
     ```
   - Use multi-stage builds:  
     ```dockerfile
     FROM node:16 AS builder
     WORKDIR /app
     COPY package.json .
     RUN npm install
     FROM node:16-slim
     COPY --from=builder /app /app
     ```
   - Remove unnecessary files:  
     ```dockerfile
     RUN rm -rf /app/tests
     ```
   - Rebuild with cache:  
     ```bash
     docker build --cache-from my_image:latest -t my_image .
     ```
   - **Key Considerations**: Leverage layer caching by ordering commands efficiently. Use multi-stage builds to reduce image size. Minimize copied files.

33. **A container built from a custom image is missing a critical dependency. How would you debug and fix this?**  
   - Check container logs:  
     ```bash
     docker logs <container_id>
     ```
   - Inspect Dockerfile for missing dependency:  
     ```bash
     cat Dockerfile
     ```
   - Add dependency to Dockerfile:  
     ```dockerfile
     RUN apt-get update && apt-get install -y <dependency>
     ```
   - Rebuild and run:  
     ```bash
     docker build -t my_image .
     docker run my_image
     ```
   - **Key Considerations**: Verify dependency requirements in application logs. Update Dockerfile to include missing packages. Test container after rebuild.

34. **You need to reduce the size of a Docker image. What strategies would you use?**  
   - Use a smaller base image:  
     ```dockerfile
     FROM alpine:latest
     ```
   - Clean up after installations:  
     ```dockerfile
     RUN apt-get update && apt-get install -y <package> && apt-get clean && rm -rf /var/lib/apt/lists/*
     ```
   - Use multi-stage builds:  
     ```dockerfile
     FROM node:16 AS builder
     WORKDIR /app
     COPY package.json .
     RUN npm install
     FROM node:16-slim
     COPY --from=builder /app /app
     ```
   - Remove unnecessary files:  
     ```dockerfile
     RUN rm -rf /app/unneeded_files
     ```
   - **Key Considerations**: Choose lightweight base images (e.g., alpine). Minimize layers by combining commands. Use multi-stage builds to exclude build artifacts.

35. **A developer reports that a Docker image is pulling an outdated version from a registry. How would you ensure the latest version is used?**  
   - Pull latest image:  
     ```bash
     docker pull my_image:latest
     ```
   - Remove cached images:  
     ```bash
     docker image rm my_image:latest
     ```
   - Verify image tag:  
     ```bash
     docker images my_image
     ```
   - Run container with latest image:  
     ```bash
     docker run my_image:latest
     ```
   - **Key Considerations**: Always specify explicit tags (avoid `latest` for production). Clear local cache to force pull. Verify registry credentials.

36. **A Docker build fails due to a network timeout during package installation. How would you troubleshoot this?**  
   - Check network connectivity:  
     ```bash
     ping archive.ubuntu.com
     ```
   - Use a different mirror:  
     ```dockerfile
     RUN echo "deb http://mirror.example.com/ubuntu focal main" > /etc/apt/sources.list
     ```
   - Retry build with increased timeout:  
     ```bash
     docker build --build-arg APT_TIMEOUT=300 -t my_image .
     ```
   - Use cached packages if available:  
     ```bash
     docker build --no-cache -t my_image .
     ```
   - **Key Considerations**: Verify host network stability. Use reliable package mirrors. Consider caching dependencies locally.

37. **You suspect a Docker image contains vulnerabilities. How would you scan and address them?**  
   - Scan image with Trivy:  
     ```bash
     trivy image my_image:latest
     ```
   - Update base image in Dockerfile:  
     ```dockerfile
     FROM ubuntu:20.04
     RUN apt-get update && apt-get upgrade -y
     ```
   - Rebuild image:  
     ```bash
     docker build -t my_image:latest .
     ```
   - Retest image:  
     ```bash
     trivy image my_image:latest
     ```
   - **Key Considerations**: Use vulnerability scanners like Trivy or Clair. Keep base images updated. Rebuild and retest after updates.

38. **A container fails to start because the base image is no longer available in the registry. How would you handle this?**  
   - Check local images:  
     ```bash
     docker images
     ```
   - Use a local copy if available:  
     ```bash
     docker tag my_image:backup my_image:latest
     ```
   - Find alternative image:  
     ```bash
     docker search <image_name>
     ```
   - Update Dockerfile with new base image:  
     ```dockerfile
     FROM new_image:latest
     ```
   - Rebuild image:  
     ```bash
     docker build -t my_image .
     ```
   - **Key Considerations**: Maintain local backups of critical images. Use specific tags to avoid registry issues. Test alternative images for compatibility.

39. **You need to create a multi-architecture Docker image. How would you approach this?**  
   - Use buildx:  
     ```bash
     docker buildx create --use
     ```
   - Build multi-arch image:  
     ```bash
     docker buildx build --platform linux/amd64,linux/arm64 -t my_image:multiarch --push .
     ```
   - Verify image:  
     ```bash
     docker buildx imagetools inspect my_image:multiarch
     ```
   - **Key Considerations**: Ensure buildx is installed. Specify all target architectures. Push to registry for distribution.

40. **A Dockerfile is producing inconsistent builds across different environments. How would you diagnose and resolve this?**  
   - Check Dockerfile for non-deterministic commands:  
     ```bash
     cat Dockerfile
     ```
   - Pin versions in Dockerfile:  
     ```dockerfile
     FROM node:16.13.2
     RUN npm install express@4.17.1
     ```
   - Use build cache consistently:  
     ```bash
     docker build --no-cache -t my_image .
     ```
   - Standardize build environment:  
     ```bash
     docker run -it <builder_image> bash
     ```
   - **Key Considerations**: Pin all package versions to ensure reproducibility. Avoid external dependencies with variable outputs. Use consistent build environments.

---

## 5. Linux System Administration

41. **A Docker daemon fails to start with the error: "cannot open /var/run/docker.pid." How would you troubleshoot this?**

   ```bash
   # Check if docker.pid exists and remove if stale
   ls -l /var/run/docker.pid
   rm -f /var/run/docker.pid

   # Verify Docker service status
   systemctl status docker

   # Attempt to start Docker
   systemctl start docker

   # Check for errors in Docker logs
   journalctl -u docker -b
   ```

   **Key Considerations**:
   - Ensure no other process is using the Docker socket.
   - Verify Docker service permissions and ownership of `/var/run/docker.pid`.
   - Check for disk space issues on the host.

42. **The Docker service is consuming excessive memory on the host. How would you investigate and resolve this?**

   ```bash
   # Check Docker service memory usage
   top -p $(pidof dockerd)

   # Inspect memory usage by containers
   docker stats --no-stream

   # Identify high-memory containers
   docker ps -q | xargs docker inspect --format '{{.Id}} {{.State.Pid}} {{.Name}}' | while read id pid name; do
       cat /proc/$pid/status | grep VmRSS
   done

   # Set memory limits on a container (e.g., 512MB)
   docker update --memory 512m <container_id>
   ```

   **Key Considerations**:
   - Monitor memory leaks in containerized applications.
   - Adjust Docker daemon configuration (`/etc/docker/daemon.json`) for resource limits.
   - Ensure swap is enabled if memory is constrained.

43. **A Linux server running Docker is running out of inodes. How would you identify and fix this issue?**

   ```bash
   # Check inode usage
   df -i

   # Find directories with high inode usage
   find / -xdev -printf '%h\n' | sort | uniq -c | sort -k 1 -nr | head

   # Remove unused Docker images, containers, and volumes
   docker system prune -a --volumes

   # Check for excessive small files in Docker storage
   du -sh /var/lib/docker
   ```

   **Key Considerations**:
   - Regularly clean up unused Docker objects to prevent inode exhaustion.
   - Verify Docker storage driver compatibility (e.g., overlay2).
   - Monitor `/var/lib/docker` for excessive temporary files.

44. **You notice high I/O wait times on a Linux server running multiple containers. How would you diagnose the cause?**

   ```bash
   # Check I/O wait times
   iostat -x 1

   # Identify processes causing high I/O
   iotop -o

   # Inspect container I/O usage
   docker stats --no-stream

   # Check disk usage by containers
   docker ps -q | xargs -I {} sh -c 'echo Container: {}; cat /sys/fs/cgroup/blkio/docker/{}/blkio.throttle.io_service_bytes'
   ```

   **Key Considerations**:
   - Identify if a specific container or process is causing I/O bottlenecks.
   - Consider using `ionice` to prioritize I/O for critical processes.
   - Evaluate storage driver performance (e.g., overlay2 vs. devicemapper).

45. **A Linux process is preventing a container from starting due to a port conflict. How would you identify and resolve it?**

   ```bash
   # Identify process using the port (e.g., port 80)
   netstat -tulnp | grep :80
   lsof -i :80

   # Kill the conflicting process
   kill -9 <pid>

   # Start the container
   docker run -d -p 80:80 <image_name>
   ```

   **Key Considerations**:
   - Verify if the port is used by another container or host process.
   - Use `docker ps` to check for running containers on the same port.
   - Consider reassigning ports if killing the process is not an option.

46. **The Docker daemon is not responding to commands. How would you restart it without impacting running containers?**

   ```bash
   # Check Docker daemon status
   systemctl status docker

   # Reload Docker daemon without stopping containers
   systemctl reload docker

   # If reload fails, restart gracefully
   systemctl restart docker

   # Verify containers are still running
   docker ps
   ```

   **Key Considerations**:
   - Docker daemon reload typically preserves running containers.
   - Ensure no critical operations are running before restarting.
   - Check logs (`journalctl -u docker`) for daemon issues.

47. **A Linux server running Docker has high load averages. How would you determine if Docker is the cause?**

   ```bash
   # Check system load
   uptime

   # Inspect resource usage by Docker processes
   top -p $(pidof dockerd)

   # Check container resource usage
   docker stats --no-stream

   # Identify high-load processes
   ps aux --sort=-%cpu | head

   # Check for I/O or memory bottlenecks
   iostat -x 1
   free -m
   ```

   **Key Considerations**:
   - Correlate high load with Docker-specific processes or containers.
   - Monitor for resource contention (CPU, memory, I/O).
   - Consider limiting container resources using `--cpu-shares` or `--memory`.

48. **You receive an alert that the root filesystem on a Docker host is full. How would you free up space?**

   ```bash
   # Check disk usage
   df -h /

   # Identify large files/directories
   du -ah / | sort -rh | head -n 10

   # Remove unused Docker objects
   docker system prune -a --volumes

   # Clean up dangling images
   docker image prune -f

   # Remove old logs
   truncate -s 0 /var/log/*.log
   ```

   **Key Considerations**:
   - Avoid deleting critical Docker data in `/var/lib/docker`.
   - Rotate logs using `logrotate` to prevent future issues.
   - Check for large container logs (`docker logs <container_id>`).

49. **A container is unable to start due to insufficient permissions on the host. How would you troubleshoot this?**

   ```bash
   # Check Docker logs for permission errors
   docker logs <container_id>

   # Verify permissions on Docker socket
   ls -l /var/run/docker.sock

   # Add user to docker group (if applicable)
   usermod -aG docker <username>

   # Check SELinux/AppArmor policies
   getenforce
   aa-status

   # Run container with adjusted permissions
   docker run --user $(id -u):$(id -g) <image_name>
   ```

   **Key Considerations**:
   - Ensure the user has access to Docker socket or relevant files.
   - Check for restrictive SELinux/AppArmor policies.
   - Avoid running containers as root unless necessary.

50. **A Linux kernel panic occurs on a server running Docker. How would you investigate the cause?**

   ```bash
   # Check system logs for kernel panic details
   cat /var/log/messages | grep -i panic
   journalctl -b -1

   # Inspect Docker daemon logs
   journalctl -u docker -b -1

   # Check for kernel crash dumps
   ls -l /var/crash

   # Analyze crash dump (if available)
   crash /var/crash/*/vmcore /usr/lib/debug/lib/modules/$(uname -r)/vmlinux
   ```

   **Key Considerations**:
   - Verify if Docker storage driver or networking caused the panic.
   - Check for recent kernel updates or module conflicts.
   - Enable `kdump` for detailed crash analysis.

---

## 6. Docker Security

51. **A container is running as root, which is against security best practices. How would you modify it to run as a non-root user?**

   ```dockerfile
   # Dockerfile example
   FROM node:16
   RUN useradd -m appuser
   USER appuser
   COPY . /app
   WORKDIR /app
   CMD ["node", "app.js"]
   ```

   ```bash
   # Run container as non-root user
   docker run --user appuser <image_name>

   # Rebuild image
   docker build -t <image_name> .
   ```

   **Key Considerations**:
   - Ensure the application supports non-root execution.
   - Verify file permissions for the non-root user.
   - Avoid modifying existing images; rebuild with a non-root user.

52. **You suspect a container has been compromised. What steps would you take to investigate and mitigate the issue?**

   ```bash
   # Inspect container processes
   docker top <container_id>

   # Check container logs
   docker logs <container_id>

   # Scan container for malware
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <image_name>

   # Isolate container by pausing it
   docker pause <container_id>

   # Remove compromised container
   docker rm -f <container_id>
   ```

   **Key Considerations**:
   - Quarantine the container to prevent further damage.
   - Check network activity for unauthorized connections.
   - Rebuild the image from a trusted source after mitigation.

53. **A Docker image contains sensitive credentials in its layers. How would you identify and remove them?**

   ```bash
   # Inspect image layers
   docker history <image_name>

   # Scan image for secrets
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <image_name>

   # Rebuild image without credentials
   # Dockerfile example
   FROM node:16
   COPY . /app
   WORKDIR /app
   # Use environment variables or secrets
   CMD ["node", "app.js"]
   ```

   **Key Considerations**:
   - Avoid hardcoding credentials in Dockerfiles.
   - Use Docker secrets or environment variables for sensitive data.
   - Squash image layers to remove sensitive data from history.

54. **A container is bypassing security policies and accessing restricted resources. How would you diagnose and fix this?**

   ```bash
   # Check container capabilities
   docker inspect <container_id> | grep CapAdd

   # Check for privileged mode
   docker inspect <container_id> | grep Privileged

   # Restrict capabilities
   docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE <image_name>

   # Enable AppArmor/SELinux
   docker run --security-opt apparmor=docker-default <image_name>
   ```

   **Key Considerations**:
   - Verify if the container runs in privileged mode or with excessive capabilities.
   - Apply AppArmor/SELinux profiles to enforce restrictions.
   - Audit container configuration for security misconfigurations.

55. **You need to restrict a container’s access to the host filesystem. How would you configure this?**

   ```bash
   # Run container with read-only filesystem
   docker run --read-only <image_name>

   # Mount specific volumes as read-only
   docker run -v /host/path:/container/path:ro <image_name>

   # Use tmpfs for temporary storage
   docker run --tmpfs /tmp <image_name>
   ```

   **Key Considerations**:
   - Avoid mounting sensitive host directories (e.g., `/etc`, `/root`).
   - Use read-only mounts for non-writable data.
   - Combine with user namespaces for additional isolation.

56. **A security scan reports that a container is using a vulnerable version of a library. How would you update it?**

   ```bash
   # Scan image for vulnerabilities
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <image_name>

   # Update Dockerfile to use latest base image
   # Dockerfile example
   FROM node:16
   RUN apt-get update && apt-get upgrade -y
   COPY . /app
   CMD ["node", "app.js"]

   # Rebuild and push updated image
   docker build -t <image_name>:latest .
   docker push <image_name>:latest
   ```

   **Key Considerations**:
   - Regularly update base images to include security patches.
   - Automate vulnerability scans in CI/CD pipelines.
   - Test updated images for compatibility before deployment.

57. **A container is making unauthorized network requests. How would you detect and block this behavior?**

   ```bash
   # Monitor container network activity
   docker network inspect <network_name>
   tcpdump -i docker0

   # Apply network policy to block outbound traffic
   iptables -A OUTPUT -s <container_ip> -j DROP

   # Run container with no network access
   docker run --network none <image_name>
   ```

   **Key Considerations**:
   - Use network namespaces to isolate container traffic.
   - Implement firewall rules (e.g., `iptables`, `nftables`) for fine-grained control.
   - Audit container network configurations for open ports.

58. **You need to implement role-based access control for Docker. How would you set this up?**

   ```bash
   # Create user and group for Docker access
   useradd -m dockeruser
   groupadd docker
   usermod -aG docker dockeruser

   # Restrict Docker socket access
   chown root:docker /var/run/docker.sock
   chmod 660 /var/run/docker.sock

   # Use Docker contexts for remote access control
   docker context create restricted --docker host=unix:///var/run/docker.sock
   ```

   **Key Considerations**:
   - Limit `docker` group membership to trusted users.
   - Use TLS for remote Docker API access.
   - Regularly audit user permissions and Docker socket access.

59. **A container is running with excessive privileges. How would you reduce its capabilities?**

   ```bash
   # Check current capabilities
   docker inspect <container_id> | grep CapAdd

   # Run container with minimal capabilities
   docker run --cap-drop=ALL --cap-add=CHOWN --cap-add=SETUID <image_name>

   # Verify capabilities inside container
   docker exec <container_id> capsh --print
   ```

   **Key Considerations**:
   - Drop all capabilities unless explicitly needed.
   - Test application functionality after reducing capabilities.
   - Combine with non-root user for enhanced security.

60. **You suspect a Docker registry is serving malicious images. How would you verify and protect against this?**

   ```bash
   # Scan image for vulnerabilities
   docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image <registry>/<image_name>

   # Verify image signature
   docker trust inspect <registry>/<image_name>

   # Pull only signed images
   docker pull --disable-content-trust=false <registry>/<image_name>

   # Configure private registry with authentication
   docker login <registry>
   ```

   **Key Considerations**:
   - Use trusted registries (e.g., Docker Hub, private registries).
   - Enable Docker Content Trust (DCT) for image signing.
   - Regularly scan images and enforce signature verification.

---

## (Out of Scope) 7. Docker Orchestration (Docker Compose/Swarm)
These questions focus on troubleshooting Docker Compose and Docker Swarm setups.

61. **A Docker Compose application fails to start with a "service not found" error. How would you troubleshoot this?**
62. **A Docker Swarm service is stuck in a "pending" state. What could be the cause, and how would you fix it?**
63. **A Docker Compose file defines a multi-container application, but one service is not communicating with another. How would you debug this?**
64. **A Swarm service is not scaling as expected. How would you investigate and resolve this?**
65. **A Docker Compose application is not using the specified environment variables. How would you troubleshoot this?**
66. **A Swarm node is marked as "Down" and not accepting tasks. How would you diagnose and bring it back online?**
67. **You need to update a Docker Swarm service without downtime. How would you perform a rolling update?**
68. **A Docker Compose application is crashing due to resource limits. How would you adjust the configuration?**
69. **A Swarm service is not load balancing traffic correctly. How would you troubleshoot this?**
70. **A Docker Compose file is producing errors during startup. How would you validate and fix the file?**

## 8. Performance and Monitoring

71. **A Dockerized application is experiencing high latency. How would you use monitoring tools to identify the bottleneck?**  
   - **Commands**:  
     ```bash
     docker stats
     top -p $(pidof dockerd)
     iostat -x 1
     netstat -i
     ```
   - **Steps**: Check container resource usage with `docker stats`. Monitor host CPU/memory with `top`. Use `iostat` for disk I/O bottlenecks. Check network interfaces with `netstat`.  
   - **Key Considerations**: Identify if latency is due to CPU, memory, disk, or network. Cross-check container metrics with host metrics to pinpoint the bottleneck.

72. **You notice that a container’s memory usage is spiking periodically. What tools would you use to monitor and diagnose this?**  
   - **Commands**:  
     ```bash
     docker stats container_name
     docker inspect container_name | grep Memory
     smem -k -p -c pss,uss
     ```
   - **Steps**: Use `docker stats` to monitor memory spikes. Check container memory limits with `docker inspect`. Use `smem` to analyze process memory usage inside the container.  
   - **Key Considerations**: Verify if spikes correlate with application activity. Check for memory leaks or misconfigured limits.

73. **A Linux server running Docker is under heavy load. How would you use tools like `top`, `htop`, or `sar` to analyze performance?**  
   - **Commands**:  
     ```bash
     top
     htop
     sar -u 1 5
     sar -r 1 5
     ```
   - **Steps**: Run `top` or `htop` to identify high-CPU/memory processes. Use `sar -u` for CPU usage trends and `sar -r` for memory usage over time.  
   - **Key Considerations**: Look for Docker daemon or container processes consuming resources. Check historical data with `sar` for patterns.

74. **A containerized application is slow to respond. How would you use Docker stats and Linux tools to pinpoint the issue?**  
   - **Commands**:  
     ```bash
     docker stats container_name
     docker top container_name
     iostat -x 1
     netstat -tunap
     ```
   - **Steps**: Check container resource usage with `docker stats`. List processes inside the container with `docker top`. Monitor disk I/O with `iostat`. Check network connections with `netstat`.  
   - **Key Considerations**: Determine if slowness is due to resource contention, disk I/O, or network issues. Correlate with application logs.

75. **You need to set up monitoring for a Dockerized application. What tools and metrics would you use?**  
   - **Commands**:  
     ```bash
     docker run -d --name prometheus -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
     docker run -d --name grafana -p 3000:3000 grafana/grafana
     ```
   - **Steps**: Deploy Prometheus to collect metrics (CPU, memory, I/O, network). Use Grafana for visualization. Monitor metrics like container_cpu_usage_seconds_total, container_memory_usage_bytes, and network I/O.  
   - **Key Considerations**: Ensure Prometheus scrapes Docker daemon and container metrics. Configure alerts for thresholds.

76. **A container is consuming excessive disk I/O. How would you use Linux tools like `iostat` or `iotop` to investigate?**  
   - **Commands**:  
     ```bash
     iostat -x 1
     iotop -o -P
     docker inspect container_name | grep Mounts
     ```
   - **Steps**: Use `iostat -x` to check disk I/O rates. Run `iotop -o -P` to identify processes causing high I/O. Verify container volume mounts with `docker inspect`.  
   - **Key Considerations**: Check if I/O is due to container processes or host issues. Consider optimizing application I/O patterns.

77. **A Docker host is experiencing network bottlenecks. How would you use tools like `iftop` or `nload` to diagnose this?**  
   - **Commands**:  
     ```bash
     iftop -i eth0
     nload eth0
     docker network inspect bridge
     ```
   - **Steps**: Use `iftop` to monitor network traffic by interface. Run `nload` for real-time bandwidth usage. Check container network configuration with `docker network inspect`.  
   - **Key Considerations**: Identify high-traffic containers or external connections. Check for misconfigured network policies.

78. **You need to benchmark a containerized application’s performance. How would you approach this on a Linux system?**  
   - **Commands**:  
     ```bash
     docker run -it container_name stress --cpu 2 --io 1 --vm 1 --vm-bytes 128M --timeout 60s
     docker stats container_name
     perf stat -p $(docker top container_name | awk 'NR>1 {print $2}')
     ```
   - **Steps**: Run `stress` inside the container to simulate load. Monitor with `docker stats`. Use `perf stat` to collect CPU/memory metrics for container processes.  
   - **Key Considerations**: Benchmark under realistic workloads. Compare results against baseline performance.

79. **A container is intermittently crashing. How would you set up logging and monitoring to capture the cause?**  
   - **Commands**:  
     ```bash
     docker logs container_name --follow
     docker events --filter 'container=container_name'
     docker run -d --name promtail -v /var/log:/var/log loki/promtail -config.file=/etc/promtail/promtail.yml
     ```
   - **Steps**: Stream logs with `docker logs`. Monitor container events with `docker events`. Set up Promtail to send logs to Loki for analysis.  
   - **Key Considerations**: Ensure logs capture crash details. Correlate events with system resource usage.

80. **You want to optimize a container’s CPU usage. What tools and techniques would you use to profile it?**  
   - **Commands**:  
     ```bash
     docker stats container_name
     docker top container_name
     perf stat -p $(docker top container_name | awk 'NR>1 {print $2}')
     docker update --cpus="0.5" container_name
     ```
   - **Steps**: Monitor CPU usage with `docker stats` and `docker top`. Profile processes with `perf stat`. Limit container CPU with `docker update`.  
   - **Key Considerations**: Identify CPU-intensive processes. Adjust CPU limits to balance performance and resource usage.

## 9. Logging and Debugging

81. **A containerized application is producing no logs. How would you configure logging and verify it’s working?**  
   - **Commands**:  
     ```bash
     docker inspect container_name | grep LogConfig
     docker run --log-driver=json-file --log-opt max-size=10m container_name
     docker logs container_name
     ```
   - **Steps**: Check logging driver with `docker inspect`. Restart container with `json-file` driver and size limits. Verify logs with `docker logs`.  
   - **Key Considerations**: Ensure application writes to stdout/stderr. Check log driver compatibility.

82. **You need to collect logs from multiple containers for analysis. How would you set this up?**  
   - **Commands**:  
     ```bash
     docker run -d --name loki -p 3100:3100 grafana/loki
     docker run -d --name promtail -v /var/lib/docker/containers:/var/lib/docker/containers loki/promtail -config.file=/etc/promtail/promtail.yml
     ```
   - **Steps**: Deploy Loki for log aggregation. Configure Promtail to collect logs from container log files. Query logs via Grafana.  
   - **Key Considerations**: Ensure Promtail has access to container log directories. Configure retention policies.

83. **A container is crashing, but the logs are incomplete. How would you increase log verbosity?**  
   - **Commands**:  
     ```bash
     docker inspect container_name | grep LogConfig
     docker run --log-driver=json-file --log-opt max-size=50m container_name --verbose
     docker logs container_name
     ```
   - **Steps**: Check current log config with `docker inspect`. Restart container with verbose application flags and larger log size. Review logs with `docker logs`.  
   - **Key Considerations**: Verify application supports verbose logging. Monitor log storage usage.

84. **A Linux process inside a container is generating errors. How would you use `strace` or `gdb` to debug it?**  
   - **Commands**:  
     ```bash
     docker exec -it container_name apt-get update && apt-get install -y strace gdb
     docker exec -it container_name strace -p $(pgrep process_name)
     docker exec -it container_name gdb --pid=$(pgrep process_name)
     ```
   - **Steps**: Install `strace` and `gdb` in the container. Use `strace` to trace system calls or `gdb` to attach to the process.  
   - **Key Considerations**: Ensure container has debugging tools. Avoid impacting production processes.

85. **A container’s logs are being written to stdout but not captured. How would you redirect them to a file?**  
   - **Commands**:  
     ```bash
     docker run --log-driver=json-file --log-opt max-size=10m -v /host/path:/logs container_name
     docker exec container_name sh -c 'echo "log output" > /logs/app.log'
     ```
   - **Steps**: Configure `json-file` driver with volume mount. Redirect application output to a file in the mounted volume.  
   - **Key Considerations**: Ensure volume permissions allow writing. Monitor log file growth.

86. **You suspect a container is leaking memory. How would you use Linux tools like `valgrind` to confirm this?**  
   - **Commands**:  
     ```bash
     docker exec -it container_name apt-get update && apt-get install -y valgrind
     docker exec -it container_name valgrind --leak-check=full --log-file=/tmp/valgrind.log process_name
     ```
   - **Steps**: Install `valgrind` in the container. Run `valgrind` with leak-check on the suspected process. Review output in `/tmp/valgrind.log`.  
   - **Key Considerations**: Ensure application is compiled with debugging symbols. Account for `valgrind` performance overhead.

87. **A containerized application is producing cryptic error messages. How would you extract more context from the logs?**  
   - **Commands**:  
     ```bash
     docker logs container_name --tail 100
     docker inspect container_name | grep Env
     docker exec container_name cat /path/to/config
     ```
   - **Steps**: Review recent logs with `docker logs`. Check environment variables with `docker inspect`. Inspect application config files for context.  
   - **Key Considerations**: Correlate errors with config or environment settings. Enable debug mode if available.

88. **You need to centralize logs from multiple Docker hosts. What solutions would you implement?**  
   - **Commands**:  
     ```bash
     docker run -d --name loki -p 3100:3100 grafana/loki
     docker run -d --name promtail -v /var/lib/docker/containers:/var/lib/docker/containers loki/promtail -config.file=/etc/promtail/promtail.yml
     ```
   - **Steps**: Deploy Loki on a central server. Configure Promtail on each host to forward container logs to Loki. Use Grafana to query logs.  
   - **Key Considerations**: Ensure network connectivity between hosts and Loki. Secure log transmission with TLS.

89. **A container is failing silently with no logs. How would you enable debugging to capture the failure?**  
   - **Commands**:  
     ```bash
     docker run --log-driver=json-file --log-opt max-size=10m container_name --debug
     docker events --filter 'container=container_name'
     docker logs container_name
     ```
   - **Steps**: Restart container with debug flag and `json-file` driver. Monitor events with `docker events`. Check logs with `docker logs`.  
   - **Key Considerations**: Verify application supports debug mode. Check for suppressed logs in config.

90. **A Linux server’s journald logs are consuming excessive disk space. How would you manage and rotate them?**  
   - **Commands**:  
     ```bash
     journalctl --disk-usage
     journalctl --vacuum-size=500M
     sudo vi /etc/systemd/journald.conf
     # Set: SystemMaxUse=500M
     sudo systemctl restart systemd-journald
     ```
   - **Steps**: Check log size with `journalctl --disk-usage`. Reduce size with `journalctl --vacuum-size`. Edit `/etc/systemd/journald.conf` to set size limits. Restart journald.  
   - **Key Considerations**: Balance log retention with disk space. Monitor after changes to ensure logs are captured.

## 10. Disaster Recovery and High Availability
These questions focus on recovering from failures and ensuring high availability in Docker and Linux environments.

91. **A Docker host crashes, and containers are not restarting automatically. How would you configure auto-recovery?**

   **Solution**: Configure containers with a restart policy and ensure the Docker daemon starts on boot.
   ```bash
   # Enable Docker to start on boot
   sudo systemctl enable docker

   # Set restart policy for a container
   docker run --restart=always -d <image_name>

   # Update existing container with restart policy
   docker update --restart=always <container_id>
   ```

   **Key Considerations**:
   - Use `always` for critical containers, `unless-stopped` for manual control, or `on-failure` for error-based restarts.
   - Verify sufficient host resources (CPU, memory) to support restarts.
   - Ensure Docker service is enabled (`systemctl is-enabled docker`).
   - Check container logs (`docker logs <container_id>`) for restart failures.

92. **A critical containerized application goes down during a host failure. How would you restore it quickly?**

   **Solution**: Use Docker Compose or stored container configuration to redeploy.
   ```bash
   # If using Docker Compose
   docker-compose -f docker-compose.yml up -d

   # Recreate from known configuration
   docker run -d --name <container_name> -p <port>:<port> -v <volume> <image_name>

   # Check container status
   docker ps -a
   ```

   **Key Considerations**:
   - Store Docker Compose files or run commands in version control.
   - Ensure images are available in a registry or local cache.
   - Verify volume data integrity before restarting.
   - Test restoration process regularly to minimize downtime.

93. **You need to back up a Dockerized database. How would you implement a backup strategy?**

   **Solution**: Create a script to back up the database and store it securely.
   ```bash
   # Example for a MySQL container
   docker exec <mysql_container> mysqldump -u root -p<password> --all-databases > backup.sql

   # Copy backup to a remote location
   scp backup.sql user@remote_host:/backups/

   # Schedule with cron
   echo "0 2 * * * docker exec <mysql_container> mysqldump -u root -p<password> --all-databases > /backups/backup_$(date +\%F).sql" | crontab -
   ```

   **Key Considerations**:
   - Use container-specific backup tools (e.g., `pg_dump` for PostgreSQL).
   - Encrypt backups for sensitive data (`gpg -c backup.sql`).
   - Store backups off-site or in cloud storage (e.g., AWS S3).
   - Test restoration (`docker exec <mysql_container> mysql < backup.sql`) to ensure backups are valid.

94. **A Docker Swarm cluster loses a manager node. How would you recover the cluster?**

   **Solution**: Promote a worker node or restore from a backup.
   ```bash
   # Check Swarm status
   docker node ls

   # Promote a worker to manager
   docker node promote <worker_node_id>

   # Restore from backup (if quorum lost)
   docker swarm init --force-new-cluster --advertise-addr <manager_ip>:2377

   # Rejoin workers
   docker swarm join --token <worker_token> <manager_ip>:2377
   ```

   **Key Considerations**:
   - Maintain an odd number of managers (3 or 5) for quorum.
   - Back up Swarm state (`/var/lib/docker/swarm/`) regularly.
   - Ensure manager nodes are on reliable hardware.
   - Verify cluster health (`docker info --format '{{.Swarm.Cluster}}'`) post-recovery.

95. **A container’s data is lost due to a volume corruption. How would you restore it from a backup?**

   **Solution**: Restore the volume from a backup and restart the container.
   ```bash
   # Stop the container
   docker stop <container_id>

   # Restore volume from backup
   tar -xzf /backups/volume_backup.tar.gz -C /var/lib/docker/volumes/<volume_name>

   # Start the container
   docker start <container_id>

   # Verify data
   docker exec <container_id> ls /data
   ```

   **Key Considerations**:
   - Regularly back up volumes (`tar -czf backup.tar.gz /var/lib/docker/volumes/<volume_name>`).
   - Verify backup integrity before restoration.
   - Use named volumes for easier management.
   - Test the restored application for data consistency.

96. **A Linux server running Docker is infected with malware. How would you isolate and recover it?**

   **Solution**: Isolate the server, clean the system, and restore containers.
   ```bash
   # Disconnect from network
   sudo ip link set eth0 down

   # Stop all containers
   docker stop $(docker ps -q)

   # Scan and remove malware (example with ClamAV)
   sudo apt install clamav
   clamscan -r / --remove

   # Update and patch system
   sudo apt update && sudo apt upgrade -y

   # Restart Docker and containers
   sudo systemctl restart docker
   docker start $(docker ps -a -q)
   ```

   **Key Considerations**:
   - Use a firewall to block malicious traffic (`ufw deny out to <malicious_ip>`).
   - Scan container images for vulnerabilities (`docker scan <image_name>`).
   - Restore from clean backups if necessary.
   - Monitor system logs (`journalctl -u docker`) for suspicious activity.

97. **You need to ensure high availability for a Dockerized application. What strategies would you use?**

   **Solution**: Use Docker Swarm with replicas and health checks.
   ```bash
   # Initialize Swarm
   docker swarm init

   # Deploy service with replicas
   docker service create --name app --replicas 3 -p 80:80 <image_name>

   # Add health check to Dockerfile
   HEALTHCHECK --interval=30s --timeout=3s CMD curl -f http://localhost/ || exit 1

   # Update service with health check
   docker service update --health-cmd="curl -f http://localhost/ || exit 1" --health-interval=30s app
   ```

   **Key Considerations**:
   - Use multiple nodes for redundancy (at least 3 for Swarm).
   - Implement a load balancer (e.g., NGINX, HAProxy) for traffic distribution.
   - Monitor service health (`docker service ps app`).
   - Ensure data persistence with volumes or external storage.

98. **A Docker host’s filesystem is corrupted. How would you recover the Docker environment?**

   **Solution**: Repair the filesystem, reinstall Docker, and restore containers.
   ```bash
   # Check and repair filesystem
   sudo fsck /dev/sdX

   # Reinstall Docker
   sudo apt remove docker.io
   sudo apt install docker.io

   # Restore containers from Compose or run commands
   docker-compose -f docker-compose.yml up -d

   # Restore volumes from backup
   tar -xzf /backups/volume_backup.tar.gz -C /var/lib/docker/volumes/
   ```

   **Key Considerations**:
   - Back up `/var/lib/docker` before repairs.
   - Verify filesystem type and tools (`fsck.ext4` for ext4).
   - Test container functionality post-recovery.
   - Consider using a configuration management tool (e.g., Ansible) for consistent setups.

99. **A containerized application is experiencing frequent outages. How would you implement failover mechanisms?**

   **Solution**: Use Docker Swarm with replicas and a load balancer.
   ```bash
   # Initialize Swarm
   docker swarm init

   # Deploy service with replicas
   docker service create --name app --replicas 3 -p 80:80 <image_name>

   # Configure external load balancer (e.g., NGINX)
   cat <<EOF > /etc/nginx/conf.d/app.conf
   upstream app {
       server <node1_ip>:80;
       server <node2_ip>:80;
       server <node3_ip>:80;
   }
   server {
       listen 80;
       location / { proxy_pass http://app; }
   }
   EOF
   sudo systemctl reload nginx
   ```

   **Key Considerations**:
   - Ensure replicas are spread across nodes (`--placement-constraints`).
   - Use health checks to detect failures (`HEALTHCHECK` in Dockerfile).
   - Monitor service logs (`docker service logs app`).
   - Test failover by stopping a node (`docker node update --availability drain <node>`).

100. **You need to simulate a Docker host failure to test recovery procedures. How would you approach this?**

    **Solution**: Simulate failure by stopping the Docker service or node.
    ```bash
    # Simulate host failure by stopping Docker
    sudo systemctl stop docker

    # In Swarm, drain a node
    docker node update --availability drain <node_id>

    # Verify service failover
    docker service ps <service_name>

    # Recover by restarting Docker or rejoining node
    sudo systemctl start docker
    docker node update --availability active <node_id>
    ```

    **Key Considerations**:
    - Ensure services are replicated across multiple nodes.
    - Test backup restoration before simulation.
    - Monitor application availability during failure (`curl <app_url>`).
    - Document recovery steps and automate where possible (e.g., scripts).


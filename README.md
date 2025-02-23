# awx_devel_24_6_1howto

latest stable version of AWX in DOCKER (suitable for plugin development)

REF: https://github.com/ansible/awx/blob/24.6.1/tools/docker-compose/README.md

install OS and:
------
```
sudo dnf -y install python3 python3-pip
sudo dnf config-manager --set-enabled ol9_codeready_builder
sudo dnf -y group install "Development Tools"
```

(optional) disable DNS via DHCP (when docker image download failed via ipv6)
------

```
sudo nmcli conn modify "eth0" ipv4.ignore-auto-dns yes
sudo nmcli conn modify "eth0" ipv4.dns  "8.8.8.8,1.1.1.1"
sudo nmcli connection up "eth0"
```

addit_ref: https://github.com/vmazurukrtelecom/shell_scripts/blob/main/general_ol.sh

INSTALL ANSIBLE VIA PIP
------
```
python3 -m pip install --upgrade pip
python3 -m pip install ansible
ansible --version
```

INSTALL DOCKER
------
ref: https://docs.docker.com/engine/install/centos/

```sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf -y update
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable docker
sudo docker version
sudo systemctl start docker
sudo systemctl status docker
sudo docker version
# sudo usermod -aG docker vagrant
sudo usermod -aG docker $USER
newgrp docker
id
```
test
```
docker run hello-world
```

INSTALL DOCKER-COMPOSE
------
(v1 ! - i.e. not "docker compose")


ref: https://docs.docker.com/compose/install/standalone/
```
sudo curl -SL https://github.com/docker/compose/releases/download/v2.33.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose version
```

CLONE REPO
------

```
git clone -b 24.6.1 https://github.com/ansible/awx.git
```

PATCH DEPENDENCIES (old)
------

/requirements/requirements.txt /requirements/requirements.in
```
-django==4.2.10  # CVE-2024-24680
+django==4.2.16  # CVE-2024-24680

-sqlparse==0.4.4
+sqlparse==0.5.2
```
/tools/ansible/roles/dockerfile/templates/Dockerfile.j2
```
     openldap-devel \
     # pin to older openssl, see jira AAP-23449
-    openssl-3.0.7 \
+#    openssl-3.0.7 \
```

or
```
cd awx
wget https://raw.githubusercontent.com/vmazurukrtelecom/awx_devel_24_6_1howto/refs/heads/main/awx_dev24-6-1.patch
git apply awx_dev24-6-1.patch
git diff
```


BUILD THE IMAGE:
------
`make docker-compose-build`


example output:


<details> 


```
[vagrant@localhost awx]$ make docker-compose-build
ansible-playbook -e ansible_python_interpreter=python3.11 tools/ansible/dockerfile.yml \
        -e dockerfile_name=Dockerfile.dev \
        -e build_dev=True \
        -e receptor_image=quay.io/ansible/receptor:devel
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Render AWX Dockerfile and sources] ***********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************
ok: [localhost]

TASK [dockerfile : Create _build directory] ********************************************************************************************************************************************
ok: [localhost]

TASK [dockerfile : Render supervisor configs] ******************************************************************************************************************************************
ok: [localhost] => (item=supervisor_web.conf)
ok: [localhost] => (item=supervisor_task.conf)
ok: [localhost] => (item=supervisor_rsyslog.conf)

TASK [dockerfile : Render Dockerfile] **************************************************************************************************************************************************
ok: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

DOCKER_BUILDKIT=1 docker build \
        -f Dockerfile.dev \
        -t ghcr.io/ansible/awx_devel:HEAD \
        --build-arg BUILDKIT_INLINE_CACHE=1 \
        --cache-from=ghcr.io/ansible/awx_devel:HEAD .
[+] Building 761.9s (51/51) FINISHED                                                                                                                                     docker:default
 => [internal] load build definition from Dockerfile.dev                                                                                                                           0.0s
 => => transferring dockerfile: 7.87kB                                                                                                                                             0.0s
 => WARN: FromAsCasing: 'as' and 'FROM' keywords' casing do not match (line 8)                                                                                                     0.0s
 => [internal] load metadata for quay.io/ansible/receptor:devel                                                                                                                    1.6s
 => [internal] load metadata for quay.io/centos/centos:stream9                                                                                                                     1.7s
 => [internal] load .dockerignore                                                                                                                                                  0.0s
 => => transferring context: 56B                                                                                                                                                   0.0s
 => ERROR importing cache manifest from ghcr.io/ansible/awx_devel:HEAD                                                                                                             0.8s
 => [builder  1/10] FROM quay.io/centos/centos:stream9@sha256:f9ac4692a0505202cb05380e3c69461320358db735288c798689502d903dbe37                                                    39.5s
 => => resolve quay.io/centos/centos:stream9@sha256:f9ac4692a0505202cb05380e3c69461320358db735288c798689502d903dbe37                                                               0.0s
 => => sha256:f9ac4692a0505202cb05380e3c69461320358db735288c798689502d903dbe37 912B / 912B                                                                                         0.0s
 => => sha256:de730643c5469515148c7fa71a1e57547a566987e2a6dc00d1111facdee4bc29 505B / 505B                                                                                         0.0s
 => => sha256:5ef0f9d95ba6fc3b0dc2e87acfe6abff5e5593d403d57e57b11115a68e8ecc55 1.16kB / 1.16kB                                                                                     0.0s
 => => sha256:b4f21276d220f21482e445c053652298c3ea350bc55e9ff52ab8030aee318391 60.76MB / 60.76MB                                                                                  32.2s
 => => extracting sha256:b4f21276d220f21482e445c053652298c3ea350bc55e9ff52ab8030aee318391                                                                                          7.1s
 => [stage-1  3/34] ADD https://copr.fedorainfracloud.org/coprs/ansible/Rsyslog/repo/epel-9/ansible-Rsyslog-epel-9.repo /etc/yum.repos.d/ansible-Rsyslog-epel-9.repo               0.8s
 => [internal] load build context                                                                                                                                                  0.1s
 => => transferring context: 53.01kB                                                                                                                                               0.0s
 => FROM quay.io/ansible/receptor:devel@sha256:f9468ced85dd6bde62be42ece0a384edb510f8710df6a7b4ca3ada38e5eb8328                                                                   57.0s
 => => resolve quay.io/ansible/receptor:devel@sha256:f9468ced85dd6bde62be42ece0a384edb510f8710df6a7b4ca3ada38e5eb8328                                                              0.1s
 => => sha256:f9468ced85dd6bde62be42ece0a384edb510f8710df6a7b4ca3ada38e5eb8328 2.36kB / 2.36kB                                                                                     0.0s
 => => sha256:0a304f4bc1efd32f874bf85c0deac08aaa2de41605218ec05b714607e3a48619 1.62kB / 1.62kB                                                                                     0.0s
 => => sha256:338ab464ba548e31abeca2fc0fd45ca689f1899e98fb2355a4f8d916771b8755 88.50MB / 88.50MB                                                                                  34.4s
 => => sha256:48556440a37c578ddf2cbbc5e15013204388a6b439618bcefbaffb88e1b40fe1 8.15kB / 8.15kB                                                                                     0.0s
 => => sha256:88c230f8027b989d4cf14f952a4557eb7691f0adaf5ed86fb972b2a41e72c25d 460B / 460B                                                                                         0.5s
 => => sha256:2f2aa965ff25dd921d4668ee43d2c3574b22a4a2e79a53f3eed95b4ed9c011ff 16.38kB / 16.38kB                                                                                   0.8s
 => => sha256:3e5b7e3255822fee535c3cd4d148e48053fc6f5a1461a43774ae9cffe83f49f7 3.89kB / 3.89kB                                                                                     1.1s
 => => sha256:844f7b56bf163f9694dcd448abada9f8050f550fec3fe565af81749003fb61e3 225B / 225B                                                                                         1.4s
 => => sha256:1a6d8871693858e79a985dcf17a01366955159076ce9a218549efa017abce210 34.99MB / 34.99MB                                                                                  13.7s
 => => sha256:ed2c05d68e877a518fe1ecd907d2b4b4bae4b2a8b35a57b6661b7e193482056a 29.99MB / 29.99MB                                                                                  30.8s
 => => extracting sha256:338ab464ba548e31abeca2fc0fd45ca689f1899e98fb2355a4f8d916771b8755                                                                                         11.2s
 => => extracting sha256:88c230f8027b989d4cf14f952a4557eb7691f0adaf5ed86fb972b2a41e72c25d                                                                                          0.0s
 => => extracting sha256:2f2aa965ff25dd921d4668ee43d2c3574b22a4a2e79a53f3eed95b4ed9c011ff                                                                                          0.0s
 => => extracting sha256:3e5b7e3255822fee535c3cd4d148e48053fc6f5a1461a43774ae9cffe83f49f7                                                                                          0.0s
 => => extracting sha256:844f7b56bf163f9694dcd448abada9f8050f550fec3fe565af81749003fb61e3                                                                                          0.0s
 => => extracting sha256:1a6d8871693858e79a985dcf17a01366955159076ce9a218549efa017abce210                                                                                          7.0s
 => => extracting sha256:ed2c05d68e877a518fe1ecd907d2b4b4bae4b2a8b35a57b6661b7e193482056a                                                                                          2.9s
 => [builder  2/10] RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial                                                                                                   1.5s
 => [stage-1  3/34] ADD https://copr.fedorainfracloud.org/coprs/ansible/Rsyslog/repo/epel-9/ansible-Rsyslog-epel-9.repo /etc/yum.repos.d/ansible-Rsyslog-epel-9.repo               0.2s
 => [builder  3/10] RUN dnf -y update && dnf install -y 'dnf-command(config-manager)' &&     dnf config-manager --set-enabled crb &&     dnf -y install     iputils     gcc      108.6s
 => [stage-1  4/34] RUN dnf -y update && dnf install -y 'dnf-command(config-manager)' &&     dnf config-manager --set-enabled crb &&     dnf -y install acl     git-core     git  72.9s
 => [stage-1  5/34] RUN pip3.11 install -vv virtualenv supervisor dumb-init build                                                                                                  8.1s
 => [stage-1  6/34] RUN rm -rf /root/.cache && rm -rf /tmp/*                                                                                                                       0.4s
 => [stage-1  7/34] RUN dnf -y install     crun     gdb     gtk3     gettext     hostname     procps     alsa-lib     libX11-xcb     libXScrnSaver     iproute     strace     v  171.8s
 => [builder  4/10] RUN pip3.11 install -vv build                                                                                                                                  3.0s
 => [builder  5/10] ADD Makefile /tmp/Makefile                                                                                                                                     0.1s
 => [builder  6/10] RUN mkdir /tmp/requirements                                                                                                                                    0.3s
 => [builder  7/10] ADD requirements/requirements.txt     requirements/requirements_tower_uninstall.txt     requirements/requirements_git.txt     /tmp/requirements/               0.1s
 => [builder  8/10] RUN cd /tmp && make requirements_awx                                                                                                                         321.9s
 => [stage-1  8/34] RUN pip3.11 install -vv git+https://github.com/coderanger/supervisor-stdout.git@973ba19967cdaf46d9c1634d1675fc65b9574f6e                                       5.8s
 => [stage-1  9/34] RUN pip3.11 install -vv black setuptools-scm build                                                                                                             7.2s
 => [stage-1 10/34] RUN dnf --enablerepo=baseos-debug -y install python3-debuginfo || :                                                                                            1.0s
 => [stage-1 11/34] RUN dnf install -y epel-next-release && dnf install -y inotify-tools && dnf remove -y epel-next-release                                                       44.4s
 => [builder  9/10] ADD requirements/requirements_dev.txt /tmp/requirements                                                                                                        0.1s
 => [builder 10/10] RUN cd /tmp && make requirements_awx_dev                                                                                                                      65.2s
 => [stage-1 12/34] COPY --from=builder /var/lib/awx /var/lib/awx                                                                                                                  8.3s
 => [stage-1 13/34] RUN ln -s /var/lib/awx/venv/awx/bin/awx-manage /usr/bin/awx-manage                                                                                             0.3s
 => [stage-1 14/34] COPY --from=quay.io/ansible/receptor:devel /usr/bin/receptor /usr/bin/receptor                                                                                 0.1s
 => [stage-1 15/34] RUN openssl req -nodes -newkey rsa:2048 -keyout /etc/nginx/nginx.key -out /etc/nginx/nginx.csr         -subj "/C=US/ST=North Carolina/L=Durham/O=Ansible/OU=A  0.8s
 => [stage-1 16/34] RUN dnf install -y podman && rpm --restore shadow-utils 2>/dev/null                                                                                           19.1s
 => [stage-1 17/34] RUN sed -i -e 's|^#mount_program|mount_program|g' -e '/additionalimage.*/a "/var/lib/shared",' -e 's|^mountopt[[:space:]]*=.*$|mountopt = "nodev,fsync=0"|g'   0.3s
 => [stage-1 18/34] RUN mkdir -p /etc/containers/registries.conf.d/ && echo "unqualified-search-registries = []" >> /etc/containers/registries.conf.d/force-fully-qualified-image  0.4s
 => [stage-1 19/34] ADD tools/ansible/roles/dockerfile/files/rsyslog.conf /var/lib/awx/rsyslog/rsyslog.conf                                                                        0.1s
 => [stage-1 20/34] ADD tools/ansible/roles/dockerfile/files/wait-for-migrations /usr/local/bin/wait-for-migrations                                                                0.1s
 => [stage-1 21/34] ADD tools/ansible/roles/dockerfile/files/stop-supervisor /usr/local/bin/stop-supervisor                                                                        0.0s
 => [stage-1 22/34] ADD tools/ansible/roles/dockerfile/files/uwsgi.ini /etc/tower/uwsgi.ini                                                                                        0.1s
 => [stage-1 23/34] ADD tools/docker-compose/launch_awx.sh /usr/bin/launch_awx.sh                                                                                                  0.0s
 => [stage-1 24/34] ADD tools/docker-compose/start_tests.sh /start_tests.sh                                                                                                        0.0s
 => [stage-1 25/34] ADD tools/docker-compose/bootstrap_development.sh /usr/bin/bootstrap_development.sh                                                                            0.0s
 => [stage-1 26/34] ADD tools/docker-compose/entrypoint.sh /entrypoint.sh                                                                                                          0.0s
 => [stage-1 27/34] ADD tools/scripts/config-watcher /usr/bin/config-watcher                                                                                                       0.0s
 => [stage-1 28/34] RUN echo /awx_devel > /var/lib/awx/venv/awx/lib/python3.11/site-packages/awx.egg-link                                                                          0.3s
 => [stage-1 29/34] RUN echo /awx_devel > /var/lib/awx/venv/awx/lib/python3.11/site-packages/awx.pth                                                                               0.3s
 => [stage-1 30/34] RUN ln -sf /awx_devel/tools/docker-compose/awx-manage /usr/local/bin/awx-manage                                                                                0.4s
 => [stage-1 31/34] RUN ln -sf /awx_devel/tools/scripts/awx-python /usr/bin/awx-python                                                                                             0.4s
 => [stage-1 32/34] RUN ln -sf /awx_devel/tools/scripts/rsyslog-4xx-recovery /usr/bin/rsyslog-4xx-recovery                                                                         0.6s
 => [stage-1 33/34] RUN for dir in       /var/lib/awx       /var/lib/awx/rsyslog       /var/lib/awx/rsyslog/conf.d       /var/lib/awx/.local/share/containers/storage       /var/  0.5s
 => [stage-1 34/34] RUN for dir in       /var/lib/awx/.local       /var/lib/awx/venv       /var/lib/awx/venv/awx/bin       /var/lib/awx/venv/awx/lib/python3.11       /var/lib/aw  0.5s
 => exporting to image                                                                                                                                                           173.5s
 => => exporting layers                                                                                                                                                          173.3s
 => => preparing layers for inline cache                                                                                                                                           0.1s
 => => writing image sha256:86aa0d761d12216b1763ce600079180bac8e323a81fae989cdb996b671b45917                                                                                       0.0s
 => => naming to ghcr.io/ansible/awx_devel:HEAD                                                                                                                                    0.0s
------
 > importing cache manifest from ghcr.io/ansible/awx_devel:HEAD:
------
[vagrant@localhost awx]$ docker images
REPOSITORY                  TAG       IMAGE ID       CREATED          SIZE
ghcr.io/ansible/awx_devel   HEAD      86aa0d761d12   10 minutes ago   2.3GB
[vagrant@localhost awx]$

```

</details>


START CONTAINERS:
------
`make docker-compose`


or run docker-compose in detached mode, start the containers using the following command: ```make docker-compose COMPOSE_UP_OPTS=-d```


example output:


<details> 


```
[vagrant@localhost awx]$ make docker-compose COMPOSE_UP_OPTS=-d
ansible-playbook -e ansible_python_interpreter=python3.11 -i tools/docker-compose/inventory tools/docker-compose/ansible/sources.yml \
    -e awx_image=ghcr.io/ansible/awx_devel \
    -e awx_image_tag=HEAD \
    -e receptor_image=quay.io/ansible/receptor:devel \
    -e control_plane_node_count=1 \
    -e execution_node_count=0 \
    -e minikube_container_group=false \
    -e enable_pgbouncer=false \
    -e enable_keycloak=false \
    -e enable_ldap=false \
    -e enable_splunk=false \
    -e enable_prometheus=false \
    -e enable_grafana=false \
    -e enable_vault=false \
    -e vault_tls=false \
    -e enable_tacacs=false \
    -e enable_otel=false \
    -e enable_loki=false \
    -e install_editable_dependencies=false \
    -e pg_tls=false \


PLAY [Render AWX Dockerfile and sources] ***********************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************
ok: [localhost]

TASK [sources : Create _sources directories] *******************************************************************************************************************************************
ok: [localhost] => (item=secrets)
ok: [localhost] => (item=receptor)

TASK [sources : Detect secrets] ********************************************************************************************************************************************************
ok: [localhost] => (item=pg_password)
ok: [localhost] => (item=secret_key)
ok: [localhost] => (item=broadcast_websocket_secret)
ok: [localhost] => (item=admin_password)

TASK [sources : Generate secrets if needed] ********************************************************************************************************************************************
skipping: [localhost] => (item=pg_password)
skipping: [localhost] => (item=secret_key)
skipping: [localhost] => (item=broadcast_websocket_secret)
skipping: [localhost] => (item=admin_password)
skipping: [localhost]

TASK [sources : Include generated secrets unless they are explicitly passed in] ********************************************************************************************************
ok: [localhost] => (item=None)
ok: [localhost] => (item=None)
ok: [localhost] => (item=None)
ok: [localhost] => (item=None)
ok: [localhost]

TASK [sources : Write out SECRET_KEY] **************************************************************************************************************************************************
ok: [localhost]

TASK [sources : Find custom error pages] ***********************************************************************************************************************************************
ok: [localhost] => (item=/home/vagrant/awx/awx/static/custom_404.html)
ok: [localhost] => (item=/home/vagrant/awx/awx/static/custom_502.html)
ok: [localhost] => (item=/home/vagrant/awx/awx/static/custom_504.html)

TASK [sources : Render configuration templates] ****************************************************************************************************************************************
ok: [localhost] => (item=database.py)
changed: [localhost] => (item=local_settings.py)
ok: [localhost] => (item=websocket_secret.py)
ok: [localhost] => (item=haproxy.cfg)
ok: [localhost] => (item=nginx.conf)
ok: [localhost] => (item=nginx.locations.conf)

TASK [sources : Get OS info for sdb] ***************************************************************************************************************************************************
ok: [localhost]

TASK [sources : Get user UID] **********************************************************************************************************************************************************
ok: [localhost]

TASK [sources : Set fact with user UID] ************************************************************************************************************************************************
ok: [localhost]

TASK [sources : Set global version if not provided] ************************************************************************************************************************************
skipping: [localhost]

TASK [sources : Generate Private RSA key for signing work] *****************************************************************************************************************************
skipping: [localhost]

TASK [sources : Generate public RSA key for signing work] ******************************************************************************************************************************
skipping: [localhost]

TASK [sources : Include LDAP tasks if enabled] *****************************************************************************************************************************************
skipping: [localhost]

TASK [sources : Include vault TLS tasks if enabled] ************************************************************************************************************************************
skipping: [localhost]

TASK [sources : Iterate through ../editable_dependencies and get symlinked directories and register the paths] *************************************************************************
skipping: [localhost]

TASK [sources : Warn about empty editable_dependnecies] ********************************************************************************************************************************
skipping: [localhost]

TASK [sources : Set fact with editable_dependencies] ***********************************************************************************************************************************
skipping: [localhost]

TASK [sources : Set install_editable_dependnecies to false if no editable_dependencies are found] **************************************************************************************
skipping: [localhost]

TASK [sources : Render Docker-Compose] *************************************************************************************************************************************************
changed: [localhost]

TASK [sources : Render Receptor Config(s) for Control Plane] ***************************************************************************************************************************
changed: [localhost] => (item=1)

TASK [sources : Create Receptor Config Lock File] **************************************************************************************************************************************
changed: [localhost] => (item=1)

TASK [sources : Render Receptor Config(s) for Control Plane] ***************************************************************************************************************************
ok: [localhost] => (item=1)

TASK [sources : Render Receptor Hop Config] ********************************************************************************************************************************************
skipping: [localhost]

TASK [sources : Render Receptor Worker Config(s)] **************************************************************************************************************************************
skipping: [localhost] => (item=1)
skipping: [localhost]

TASK [sources : Render prometheus config] **********************************************************************************************************************************************
skipping: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************
localhost                  : ok=14   changed=4    unreachable=0    failed=0    skipped=13   rescued=0    ignored=0

ansible-galaxy install --ignore-certs -r tools/docker-compose/ansible/requirements.yml;
Starting galaxy collection install process
Nothing to do. All requested collections are already installed. If you want to reinstall them, consider using `--force`.
ansible-playbook -e ansible_python_interpreter=python3.11 -i tools/docker-compose/inventory tools/docker-compose/ansible/initialize_containers.yml \
    -e enable_vault=false \
    -e vault_tls=false \
    -e enable_ldap=false; \
make docker-compose-up

PLAY [Run any pre-hooks for other container] *******************************************************************************************************************************************

TASK [Initialize vault] ****************************************************************************************************************************************************************
skipping: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************
localhost                  : ok=0    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

make[1]: Entering directory '/home/vagrant/awx'
docker compose -f tools/docker-compose/_sources/docker-compose.yml  up -d --remove-orphans
WARN[0000] /home/vagrant/awx/tools/docker-compose/_sources/docker-compose.yml: the attribute `version` is obsolete, it will be ignored, please remove it to avoid potential confusion
[+] Running 13/13
 ✔ postgres Pulled                                                                                                                                                                35.8s
   ✔ 972d4b520f08 Pull complete                                                                                                                                                   28.1s
   ✔ 307d65ce577a Pull complete                                                                                                                                                   29.7s
   ✔ 910227553be5 Pull complete                                                                                                                                                   33.2s
 ✔ redis_1 Pulled                                                                                                                                                                 13.6s
   ✔ c29f5b76f736 Pull complete                                                                                                                                                    9.8s
   ✔ 5de2dd3ff2ef Pull complete                                                                                                                                                    9.9s
   ✔ 6c334acf232e Pull complete                                                                                                                                                   10.0s
   ✔ 3090e1a50a6c Pull complete                                                                                                                                                   10.3s
   ✔ f5bc47c37726 Pull complete                                                                                                                                                   11.1s
   ✔ 20eea55b3ebb Pull complete                                                                                                                                                   11.1s
   ✔ 4f4fb700ef54 Pull complete                                                                                                                                                   11.2s
   ✔ d128ccd842a6 Pull complete                                                                                                                                                   11.2s
[+] Running 5/5
 ✔ Network awx                 Created                                                                                                                                             0.3s
 ✔ Network service-mesh        Created                                                                                                                                             0.2s
 ✔ Container tools_postgres_1  Started                                                                                                                                             0.8s
 ✔ Container tools_redis_1     Started                                                                                                                                             0.8s
 ✔ Container tools_awx_1       Started                                                                                                                                            10.1s
make[1]: Leaving directory '/home/vagrant/awx'
[vagrant@localhost awx]$ docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED          STATUS          PORTS                                                                                                                                                                                                              NAMES
3594fcf1c206   ghcr.io/ansible/awx_devel:HEAD     "/entrypoint.sh laun…"   27 seconds ago   Up 17 seconds   0.0.0.0:2222->2222/tcp, 0.0.0.0:6899->6899/tcp, 0.0.0.0:7899-7999->7899-7999/tcp, 0.0.0.0:8013->8013/tcp, 0.0.0.0:8043->8043/tcp, 0.0.0.0:8080->8080/tcp, 22/tcp, 0.0.0.0:9888->9888/tcp, 0.0.0.0:3000->3001/tcp   tools_awx_1
cdc68664bc8c   quay.io/sclorg/postgresql-15-c9s   "container-entrypoin…"   27 seconds ago   Up 26 seconds   0.0.0.0:5441->5432/tcp                                                                                                                                                                                             tools_postgres_1
6759b8022161   redis:latest                       "redis-server /usr/l…"   27 seconds ago   Up 26 seconds   6379/tcp                                                                                                                                                                                                           tools_redis_1
[vagrant@localhost awx]$

```

</details>


check logs `docker logs tools_awx_1`

MAKE IU: 
------
`docker exec tools_awx_1 make clean-ui ui-devel`


example output:


<details>

```
[vagrant@localhost awx]$ docker exec tools_awx_1 make clean-ui ui-devel
rm -rf node_modules
rm -rf awx/ui/node_modules
rm -rf awx/ui/build
rm -rf awx/ui/src/locales/_build
rm -rf awx/ui/.ui-built
mkdir -p awx/ui/build/static
NODE_OPTIONS=--max-old-space-size=6144 npm --prefix awx/ui --loglevel warn --force ci
npm WARN using --force Recommended protections disabled.
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated @babel/polyfill@7.12.1: 🚨 This package has been deprecated in favor of separate inclusion of a polyfill and regenerator-runtime (when needed). See the @babel/polyfill docs (https://babeljs.io/docs/en/babel-polyfill) for more information.

added 1884 packages, and audited 1885 packages in 2m

123 packages are looking for funding
  run `npm fund` for details

43 vulnerabilities (5 low, 13 moderate, 23 high, 2 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues possible (including breaking changes), run:
  npm audit fix --force

Some issues need review, and may require choosing
a different dependency.

Run `npm audit` for details.
npm notice
npm notice New major version of npm available! 8.5.0 -> 11.1.0
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v11.1.0>
npm notice Run `npm install -g npm@11.1.0` to update!
npm notice
make[1]: Entering directory '/awx_devel'
make awx/ui/node_modules
make[2]: Entering directory '/awx_devel'
NODE_OPTIONS=--max-old-space-size=6144 npm --prefix awx/ui --loglevel warn --force ci
npm WARN using --force Recommended protections disabled.
npm WARN deprecated source-map-url@0.4.1: See https://github.com/lydell/source-map-url#deprecated
npm WARN deprecated @babel/polyfill@7.12.1: 🚨 This package has been deprecated in favor of separate inclusion of a polyfill and regenerator-runtime (when needed). See the @babel/polyfill docs (https://babeljs.io/docs/en/babel-polyfill) for more information.

added 1884 packages, and audited 1885 packages in 2m

123 packages are looking for funding
  run `npm fund` for details

43 vulnerabilities (5 low, 13 moderate, 23 high, 2 critical)

To address issues that do not require attention, run:
  npm audit fix

To address all issues possible (including breaking changes), run:
  npm audit fix --force

Some issues need review, and may require choosing
a different dependency.

Run `npm audit` for details.
make[2]: Leaving directory '/awx_devel'
python3.11 tools/scripts/compilemessages.py
/awx_devel/tools/scripts/compilemessages.py:59: DeprecationWarning: 'locale.getdefaultlocale' is deprecated and slated for removal in Python 3.15. Use setlocale(), getencoding() and getlocale() instead.
  encoding = locale.getdefaultlocale()[1] or 'ascii'
processing file django.po in /awx_devel/awx/locale/en-us/LC_MESSAGES

processing file django.po in /awx_devel/awx/locale/es/LC_MESSAGES

processing file django.po in /awx_devel/awx/locale/fr/LC_MESSAGES

processing file django.po in /awx_devel/awx/locale/ja/LC_MESSAGES

processing file django.po in /awx_devel/awx/locale/ko/LC_MESSAGES

processing file django.po in /awx_devel/awx/locale/nl/LC_MESSAGES

processing file django.po in /awx_devel/awx/locale/zh/LC_MESSAGES

npm --prefix awx/ui --loglevel warn run compile-strings

> compile-strings
> lingui compile

Compiling message catalogs…
Done!
npm --prefix awx/ui --loglevel warn run build

> build
> INLINE_RUNTIME_CHUNK=false react-scripts build

Creating an optimized production build...
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Browserslist: caniuse-lite is outdated. Please run:
  npx update-browserslist-db@latest
  Why you should do it regularly: https://github.com/browserslist/update-db#readme
Compiled with warnings.

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/cache.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/cache.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/callbackiterresult.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/callbackiterresult.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/datetime.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/datetime.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/dateutil.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/dateutil.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/datewithzone.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/datewithzone.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/helpers.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/helpers.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/index.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/index.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iter/index.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iter/index.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iter/poslist.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iter/poslist.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/easter.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/easter.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/index.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/index.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/monthinfo.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/monthinfo.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/yearinfo.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iterinfo/yearinfo.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iterresult.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iterresult.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/iterset.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/iterset.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/masks.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/masks.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/nlp/index.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/nlp/index.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/nlp/i18n.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/nlp/i18n.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/nlp/parsetext.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/nlp/parsetext.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/nlp/totext.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/nlp/totext.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/optionstostring.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/optionstostring.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/parseoptions.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/parseoptions.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/parsestring.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/parsestring.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/rrule.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/rrule.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/rruleset.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/rruleset.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/rrulestr.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/rrulestr.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/types.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/types.ts'

Failed to parse source map from '/awx_devel/awx/ui/node_modules/rrule/src/weekday.ts' file: Error: ENOENT: no such file or directory, open '/awx_devel/awx/ui/node_modules/rrule/src/weekday.ts'

Search for the keywords to learn more about each warning.
To ignore, add // eslint-disable-next-line to the line before.

File sizes after gzip:

  921.82 kB  build/static/js/main.be3da92f.js
  99.29 kB   build/static/css/main.bcaaa591.css
  61.85 kB   build/static/js/489.6fff47ab.chunk.js
  48.75 kB   build/static/js/118.55093eea.chunk.js
  45.69 kB   build/static/js/138.c73deea6.chunk.js
  44.45 kB   build/static/js/787.49d56b6e.chunk.js
  44.06 kB   build/static/js/896.7b610740.chunk.js
  43.2 kB    build/static/js/11.355f18b0.chunk.js
  42.28 kB   build/static/js/418.0a72109b.chunk.js
  30.71 kB   build/static/js/311.0bfa73ce.chunk.js
  386 B      build/static/js/979.f16bcc0c.chunk.js

The bundle size is significantly larger than recommended.
Consider reducing it with code splitting: https://goo.gl/9VhYWB
You can also analyze the project dependencies: https://goo.gl/LeUzfb

The project was built assuming it is hosted at ./.
You can control this with the homepage field in your package.json.

The build folder is ready to be deployed.

Find out more about deployment here:

  https://cra.link/deployment

touch awx/ui/.ui-built
make[1]: Leaving directory '/awx_devel'
[vagrant@localhost awx]$

```

</details>

CREATE ADMIN USER:
------
```
[vagrant@localhost awx]$ docker exec -ti tools_awx_1 awx-manage createsuperuser
Username: admin
Error: That username is already taken.
Username: root
Email address: a@a.com
Password:
Password (again):
Superuser created successfully.
[vagrant@localhost awx]$
```

OPEN UI:
------
The UI can be reached in your browser at https://localhost:8043/#/home, and the API can be found at https://localhost:8043/api/v2

RESULT EXAMPLE:
------
![image](https://github.com/user-attachments/assets/b2f5519a-92b6-479f-985e-c2e6467597b6)

![image](https://github.com/vmazurukrtelecom/awx_devel_24_6_1howto/blob/main/image.png)

addit ref: https://github.com/vmazurukrtelecom/shell_scripts/blob/main/install_awx17_OL8.sh 






[< Back to menu](../README.md)

---

# Service file

Configure service to run some script/file. Usefull for agents, servers, jobs, timers, ...

[More info on how to create one.](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6)

Basic structure
```
[Unit]
Description=Service name
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=centos
ExecStart=/usr/bin/env php /path/to/server.php

[Install]
WantedBy=multi-user.target
```

---

[< Back to menu](../README.md)

---

<p align="center">
    <b><a href="https://github.com/MGSE97" target="_blank">MGSE97</a> 🔥 2021</b>
</div>
systemd服务服务配置

```
cp ./*.service /etc/systemd/system/
```

## prometheus:
```
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
systemctl status prometheus
```
## alertmanager:
```
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
systemctl status prometheus
```


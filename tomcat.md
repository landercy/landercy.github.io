```bash
# 设置自启动
tee /etc/systemd/system/tomcat8080.service << EOF
[Unit]
Description=Tomcat for [ <SYSTEM NAME> ] listening on port [8080]
After=network.target

[Service]
Type=forking
Environment="JAVA_HOME=/data/apps/jdk"
ExecStart=/data/apps/tomcat8080/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID
User=www
Group=www

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now tomcat8080
ps -ef | grep tomcat8080

ln -s /data/apps/tomcat8080/logs tomcat8080
```

## SERVER.XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Server port="48080" shutdown="SHUTDOWN">
    <Listener className="org.apache.catalina.startup.VersionLoggerListener" />
    <Listener className="org.apache.catalina.core.AprLifecycleListener" SSLEngine="on" />
    <Listener className="org.apache.catalina.core.JreMemoryLeakPreventionListener" />
    <Listener className="org.apache.catalina.mbeans.GlobalResourcesLifecycleListener" />
    <Listener className="org.apache.catalina.core.ThreadLocalLeakPreventionListener" />

    <GlobalNamingResources>
        <Resource name="UserDatabase" auth="Container"
            type="org.apache.catalina.UserDatabase"
            description="User database that can be updated and saved"
            factory="org.apache.catalina.users.MemoryUserDatabaseFactory"
            pathname="conf/tomcat-users.xml" />
    </GlobalNamingResources>
    <Service name="Catalina">
        <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" />
        <Engine name="Catalina" defaultHost="localhost">
            <Realm className="org.apache.catalina.realm.LockOutRealm">
                <Realm className="org.apache.catalina.realm.UserDatabaseRealm"
                    resourceName="UserDatabase" />
            </Realm>
            <Host name="localhost" unpackWARs="true" autoDeploy="true" appBase="/data/web/eshop_ce">
                <Valve className="org.apache.catalina.valves.RemoteIpValve"
                    remoteIpHeader="X-Forwarded-For"
                    protocolHeader="X-Forwarded-Proto"
                    protocolHeaderHttpsValue="https" />
            </Host>
        </Engine>
    </Service>
</Server>
```

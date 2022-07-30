### How to run

1. Clone this repository

2. Run: `git submodule init`

3. Run: `git submodule update --remote` 

4. Run: `docker compose up --build -d`

-> Install latest Portainer CE at port 80:
docker run -d -p 8000:8000 -p 80:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce


Next steps:
-> Change uint8 to int on tester to see if it fixes the issue with Open5GS. => UNSUCCESS
-> Change free5GS to a newer version to see if I can connect more than 10 devices. => SUCCESS
    -> Try to update Open5GS too. => DIDN'T TRY YET
-> Test with OAI. => KINDA SUCCESS (OAI worked, connecting only 15 devices)


Important notes:
-> free5GC Web UI: admin:free5gc
-> Open5GS Web UI: admin:1423
-> InfluxDB Web UI: admin:admin1234


-> Document all my work => DOING
-> Connect/Disconnect same 10 devices multiple times => DIDN'T WORK
-> Test with multiple gNodeB (e.g.: 100 user per gNodeB) => TODO
-> Test data plane with iPerf. => TODO


Notes:
-> Using the latest stable version of the my5G Ran Tester (v1.0.0), I had a few issues during the connection of the Data Plane. The first UE was connecting sucessfully, but after the second, an error was appearing because of an error generating an ID, reusing the same previous ID, causing a conflict.
    -> To fix it, I used the code on "master" branch, as this issue was solved on this branch.

-> Running the stress test over the free5GC v3.0.6, the first 9~10 UEs connected successfully, with the Data Plane initiated, but after this, all other UEs was registering, without initialize the Data Plane.
    -> Updating the core to version v3.2.1 (latest stable version available at the moment as the test was executed), I could connect 255 devices sucessfully with Data Plane. After this, I started receiving registration error for a few devices and then the tester crashed.
-> Running it with the Open5GS (version specified at the my5G tester wiki), I could connect 255 devices sucessfully with Data Plane. The behavior was the same as with the free5GC v3.2.1.
    -> I suspect the issue is related with something inside the tester. I created an Issue on GitHub, adding some execution logs, and I'm waiting for someone to answer it.
-> Testing with OAI v1.3.0, I could connect successfully 15 devices with Data Plane and, after that, all other was registering without data plane.
    -> I tried to update to the latest stable version (v1.4.0), but the result was the same.
    -> Probably it's an issue with the core itself.

-> As I was receiving errors when I was trying to connect more than 255 devices simultaneously, I decided to test disconnecting (releasing) the UE after a few time. With this change, I could connect more than 1.000 devices, depeinding of the delays.
-> I tried to do a little change (reusing the same UE IMSI), where I was connecting 10 devices, than releasing their connections, and then repeating this a few times. It was unsuccessfully. I was receiving a lot of registration errors.


Changes:
-> TESTER:
    -> Added ANALYTICS on the source code:
        -> It's a log method to log the current epoch timestamp in nanosseconds.
        -> I'm using it inside the FSM of the UE to log every change on the UE state.
        -> I'm also using it to log when the tester starts the registration process and when the Data Plane is ready (last step).
        -> The data can be collected and parsed to CSV using an script in NodeJS.
        -> The idea is to collect a few timestamps per state to capture state transitions during the UE connection, to compare the connection speed at different core implementations.
    -> Added a new test template (test-multi-ues-in-parallel.go), it's very similar to the queue version, but starting a new connection before the previous finish.
        -> With it, we can see how the core handles multiple simultaneous connections and if an amount of UEs connected can affect new connections (increasing registration delay).
        -> A few parameters was added to configure the experiment without changing the tester code.
-> Docker composes:
    -> Updated main docker compose to run my fork of the tester, instead of the main repository version.
-> Core:
    -> Created a few repositories to adapt the latest version of the free5GC to run with the my5G tester.
-> Scripts:
    -> One repository with a few scripts to automate the experiment execution, data parsing and environment cleaning.
    -> One repository with the JS script to parse the logs.
    -> A few repositories with scripts to fill the core DB with lots of UE IMSI, to allow the tester to connect all the necessary UEs.

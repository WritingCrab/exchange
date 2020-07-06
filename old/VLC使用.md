发udp流：

```bash
./vlc.exe ./xxx.ts --sout udp://192.165.53.45:22000
```

```bash
./vlc.exe ./EMR_720p_IPP_1min.ts -vvv --loop --sout "#standard{access=udp,mux=ts,dst=192.165.53.45:22000/stream}"
```


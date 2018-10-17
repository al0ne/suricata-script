# Suricata-script

监控脚本

    #!/bin/bash
    while [ "1" ]
    do
    sleep 1
    clear
    printf "Suricata IDS 监控脚本\n\n"
    nicpacket=`ifconfig eth4|ag  -o 'RX.*(?<=packets:)(\d+)'|awk -F ':' '{print $2}'`
    printf "网卡总流量: %d\n" $nicpacket
    nicloss=`ifconfig eth4|ag  -o 'RX.*(?<=dropped:)(\d+)'|awk -F ':' '{print $4}'`
    printf "网卡丢包: %d\n" $nicloss
    ethlos=`ethtool -S eth4|ag 'error|drop' |ag -v ':\s0'`
    echo "    " $ethlos
    nicoutput=`awk -v nicloss="$nicloss" -v nicpacket="$nicpacket" 'BEGIN{printf "%.4f%%\n",(nicloss/nicpacket)*100}'`
    printf "网卡丢包率: %s\n\n" $nicoutput
    packet=`grep Suricata /proc/net/pf_ring/*eth4*|awk -F ':' '{print $1}'|xargs -i{} cat {}|ag pack|awk -F ':' '{sum+=$2};END{printf "%10.0f\n",sum}'`
    printf "Suricata总流量: %d \n" $packet
    loss=`grep Suricata /proc/net/pf_ring/*eth4*|awk -F ':' '{print $1}'|xargs -i{} cat {}|ag los|awk -F ':' '{sum+=$2};END{printf "%10.0f\n",sum}'`
    printf "Suricata丢包: %s\n" $loss
    output=`awk -v loss="$loss" -v packet="$packet" 'BEGIN{printf "%.4f%%\n",(loss/packet)*100}'`
    printf "Suricata丢包率: %s\n\n" $output
    memory=`free -mh|ag -o '(?<=cache:)\s+[\d\.]+G'|sed 's/ //g'`
    printf "内存占用: %s\n" $memory
    runtime=`stat /var/run/suricata.pid|ag -o '(?<=最近更改：)\d{4}-\d{2}-\d{2}\s[\d:]{8}'`
    echo "程序运行时间:" $runtime
    alert=`cat /data/suricata/eve.json|wc -l`
    echo "Suricata IDS产生告警:" $alert
    rules=`cat /var/lib/suricata/rules/suricata.rules|ag '^alert'|wc -l`
    printf "加载规则: %s 条\n" $rules
    done

# logcat_bash


### 목적
 
Android logcat을 편하게 사용하기.

> adb가 path에 등록되어 있어야함. app이 실행(package 검색)되어있어야 함.
~~~bash

#/bin/bash

# message를 색상별로 출력한다.
print_msg()
{
    colors=("\033[1;38;42m$1\033[0m" "\033[1;32;45m$1\033[0m" "\033[1;34;48m$1\033[0m") 
    echo -e ${colors[$2]}
}

rst=0
# 원하는 문자열인지 검사한다.
compare()
{
    DROP_WORD=(List device of devices attached)
    for sIndx in ${DROP_WORD[@]}
    do
        if [ "$1" == "$sIndx" ];
        then
            rst=0
            break
        else
            rst=2
        fi
    done
    return $rst 
}

# logcat을 캡쳐한다.
wathcing_logcat()
{
    sresult=$(adb -s $1 shell ps)  
    
    # awk의 비교분기문을 재대로 못사용해서 쓴 편법.
    pid=$(echo "$sresult" | grep "$2" | awk '{  print $2 }')
    print_msg "pid is $pid"  2

    # pipe comunication
    pfile=redirect.fifo

    rm -f $pfile
    if [[ ! -p $pfile ]] ; then
        mkfifo $pfile
    fi

    exec 6<>$pfile    

    # async logcat run
    adb -s $1 logcat -c
    adb -s $1 logcat >&6 & 

    # read fifo
    result=""
    while read line
    do
        # awk의 비교분기문을 재대로 못사용해서 쓴 편법.
        #echo "$line" | grep $pid | awk '{for (i=1; i <=NF;  i++) if (i > 1) printf("%s ", $i); printf("\n") }'
        echo "$line" | grep $pid | awk '{print $0 }' 

    done <&6

}

# main 함수
main()
{
    devicelist=$(adb devices)
    names=()
    count=0

    clear

    for a in $devicelist 
    do
        compare $a
        if [ "$rst" -ne 0 ];
        then
            print_msg "$count. <$a> is connected" 1
            names[$count]=$a
            let count+=1
        fi
    done
    echo -n "번호를 입력하세요>"
    read num
   
    if [ "$num" -lt ${#names[@]} ];
    then
        echo -n "package명을 입력하세요>"
        read packname 

        wathcing_logcat  ${names[$num]} $packname
    fi
}

main

~~~



## 실행

![](/logcat_script.gif)


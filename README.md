# scripts_and_libs
Various custom scripts and libraries

# tiny zsh ip manipulation lib
`#!/usr/bin/zsh

# return array containing individual octets for input
#  getocs 10.1.16.0
#  > 10 1 16 0
getocs() {
    [[ $@ =~ ([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3}) ]] && echo $match
}
# inverse subnet mask for start and end ip 
#   diff 10.1.16.0 10.1.17.255
#   > 0.0.1.255
ip_diff() {
    dec2ip $(( $(ip2dec $2) - $(ip2dec $1) ))
}
bin2dec() {
    echo "obase=10; ibase=2; $@" | bc
}
bin2ip() {
    [[ $@ =~ ([01]{8})([01]{8})([01]{8})([01]{8}) ]] && ip=$(bin2dec $match[1]).$(bin2dec $match[2]).$(bin2dec $match[3]).$(bin2dec $match[4]) && echo $ip
}
ip2dec () {
    ip=( $(echo $@ | sed -r 's/\./ /g') )
    printf '%d' "$(( (( ip[1] * 256 ** 3 )) + (( ip[2] * 256 ** 2)) + (( ip[3] * 256 )) + ip[4] ))"
}
ip2bin() {
    dec2bin $(ip2dec $@)
}
#invert a given subnet mask
sn2inv() {
    bin2dec $(ip2bin $@ | tr '01' '10')
}
dec2bin() {
    local dec=$@ bin=0 count=0
    while [[ $dec -gt 0 ]]
    do
        bin="$(( dec % 2 ))$bin"
        dec="$((10#$dec))"
        (( dec = dec / 2 ))
    done
    count=${#bin} && while [[ $count -le 32 ]]
    do
        bin="0$bin"
        count=${#bin}
    done
    [[ $bin =~ ([0-1]{32})[0]? ]] && bin=( $match )
    echo $bin
}
dec2ip() {
    bin2ip $(dec2bin $@)
}
# returns last address from subnet
#   last 10.1.16.30 255.255.254.0
#   > 10.1.17.255
last() {
    local first=$1 subnet=$2
    dec2ip $(( $(ip2dec $first) | $(sn2inv $subnet) ))
}
# returns first address from subnet
#   first 10.1.12.30 255.255.255.0
#   > 10.1.12.0
first() {
    local ip=$1 subnet=$2
    dec2ip $(( $(ip2dec $subnet) & $(ip2dec $ip) ))
}`

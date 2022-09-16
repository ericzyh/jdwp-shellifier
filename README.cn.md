# jdwp 命令测试脚本


## 命令

    % python ./jdwp-shellifier.py -h
        usage: jdwp-shellifier.py [-h] -t IP [-p PORT] [--break-on JAVA_METHOD]
                            [--cmd COMMAND]

        Universal exploitation script for JDWP by @_hugsy_

        optional arguments:
        -h, --help            show this help message and exit
        -t IP, --target IP    Remote target IP (default: None)
        -p PORT, --port PORT  Remote target port (default: 8000)
        --break-on JAVA_METHOD
        Specify full path to method to break on (default:
            java.net.ServerSocket.accept)
            --cmd COMMAND         Specify full path to method to break on (default:
                None)
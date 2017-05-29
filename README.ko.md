openfortivpn
============

openfortivpn은 PPP+SSL VPN 터널 서비스의 클라이언트입니다.  
이 프로그램은 pppd 프로세스를 생성하고, 게이트웨이서버와 
pppd 프로세스의 통신을 중계합니다.

이 프로그램은 Fortinet VPN과 호환됩니다.



---------
실행 예제
---------

* 간단한 VPN 연결:
  ```
  openfortivpn vpn-gateway:8443 --username=foo
  ```

* realm 인증을 이용한 VPN 연결:
  ```
  openfortivpn vpn-gateway:8443 --username=foo --realm=bar
  ```

* IP의 route 등록 및 VPN 네임서버의 `/etc/resolv.conf` 등록이 되지 않도록 연결:
  ```
  openfortivpn vpn-gateway:8443 -u foo -p bar --no-routes --no-dns
  ```

* 설정파일 이용:
  ```
  openfortivpn
  ```

  `/etc/openfortivpn/config`에는 다음의 내용이 포함되어야합니다:
  ```
  host = vpn-gateway
  port = 8443
  username = foo
  password = bar
  set-dns = 0
  set-routes = 0
  # X509 certificate sha256 sum, trust only this one!
  trusted-cert = e46d4aff08ba6914e64daa85bc6112a422fa7ce16631bff0b592a28556f993db
  ```



----
설치
----

openfortivpn은 Fedora와 Gentoo, NixOS에서는 `openfortivpn` 패키지로 제공됩니다.

다른 배포판에서는, 소스를 이용하여 빌드 및 설치를 해야합니다.

1.  빌드 의존성 설치

    * RHEL/CentOS/Fedora: `gcc` `automake` `autoconf` `openssl-devel`
    * Debian/Ubuntu: `gcc` `automake` `autoconf` `libssl-dev`
    * Arch Linux: `gcc` `automake` `autoconf` `openssl`
    * Gentoo Linux: `net-dialup/ppp`
    * openSUSE: `gcc` `automake` `autoconf` `libopenssl-devel`
    * macOS(Homebrew): `automake` `autoconf` `openssl@1.0`

    리눅스인 경우, 여러분이 직접 커널을 관리한다면, 
    다음 모듈이 컴파일 되어있어야합니다:
    ```
    CONFIG_PPP=m
    CONFIG_PPP_ASYNC=m
    ```

    macOS인 경우, 빌드 의존성 설치를 위해서 'Homebrew'를 설치해야합니다:
    ```shell
    # 'Homebrew' 설치
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

    # 의존성 설치
    brew install automake autoconf openssl@1.0
    ```


2.  빌드 및 설치.

    리눅스인 경우:
    ```shell
    aclocal && autoconf && automake --add-missing
    ./configure --prefix=/usr/local --sysconfdir=/etc
    make
    sudo make install
    ```

    macOS인 경우:
    ```shell
    export CPPFLAGS="-I/usr/local/opt/openssl/include"
    export LDFLAGS="-L/usr/local/opt/openssl/lib"
    aclocal && autoconf && automake --add-missing
    ./configure --prefix=/usr/local --sysconfdir=/etc
    make
    sudo make install
    ```



------------------
root권한으로 실행?
------------------

openfortivpn은 접속 중에 행해지는 다음 세 가지 단계에서 권한상승이 필요합니다:

* `/usr/sbin/pppd` 프로세스 생성 단계;
* VPN을 통하도록 IP route 설정 단계(터널 활성화 시);
* `/etc/resolv.conf`에 네임서버 등록 단계 (터널 활성화 시).

이러한 이유로, `sudo openfortivpn` 명령으로 실행하셔야합니다.  
sudoer가 아닌 사용자들이 실행할 수 있도록 하려면, `/etc/sudoers`에 
다음을 추가하는 것을 고려해보세요. 

예제:
`visudo -f /etc/sudoers.d/openfortivpn`
```
Cmnd_Alias  OPENFORTIVPN = /usr/bin/openfortivpn

%adm       ALL = (ALL) OPENFORTIVPN
```

**경고**: 신뢰된 사용자만 root권한으로 openfortivpn을 실행할 수 있도록 해주세요!  
[#54](https://github.com/adrienverge/openfortivpn/issues/54)의 설명대로라면,
악성 사용자가 `--pppd-plugin` 옵션과 `--pppd-log` 옵션을 이용하여 
프로그램의 행동을 우회할 수 있다고 합니다.



-------------
프로젝트 기여
-------------

자유롭게 반영 요청을 보내주세요!  

C 코딩 스타일은 반드시 다음을 따라야합니다. 
[Linux kernel Documentation/CodingStyle](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/process/coding-style.rst?id=refs/heads/master).

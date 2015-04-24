Wandbox for PHP のインストール方法とメモの兼用
==============================================

1. EPEL レポジトリのセットアップ（PHPのビルド等に必要）

	```
	# wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
        # yum localinstall epel-release-7-5.noarch.rpm
	```

1. ユーザの作成

	```
	# useradd -r -N -s /bin/nologin wandbox
	```

1. 必要なパッケージのインストール

	```
	# yum install autoconf automake bison boost boost-*.x86_64 bzip2 cmake gcc gcc-c++ git gmp-devel \
            libcap-devel libcurl-devel libevent-devel libgcrypt-devel libicu-devel libjpeg-turbo-devel libmcrypt-devel \
            libpng-devel libtidy-devel libtool-ltdl-devel libxml2-devel libxslt-devel make mariadb-devel openssl-devel \
            patch pcre-devel php-bcmath php-cli php-common php-gd php-intl php-mbstring php-xml readline-devel \
            sqlite-devel sudo tar zlib-devel
        ```

1. CATTLESHED のビルド

	```
        $ git clone --recursive https://github.com/melpon/wandbox.git wandbox
        $ pushd wandbox
        $ autoreconf
        $ ./configure --prefix=/opt/wandbox
        $ make
        $ sudo make install
        $ popd
        ```

1. KENNEL のビルド

	1. CppCMS のビルド

		```
        	$ git clone https://github.com/melpon/cppcms.git
        	$ mkdir cppcms_build
        	$ pushd cppcms_build
        	$ cmake ../cppcms/ -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/cppcms -DDISABLE_SHARED=ON \
        	    -DDISABLE_FCGI=ON -DDISABLE_SCGI=ON -DDISABLE_ICU_LOCALE=ON -DDISABLE_TCPCACHE=ON
        	$ make
        	$ sudo make install
        	$ popd
        	```

	2. CppDB のビルド

        	```
        	$ git clone https://github.com/melpon/cppdb.git
        	$ mkdir cppdb_build
        	$ pushd cppdb_build
        	$ cmake ../cppdb/ -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/cppdb -DDISABLE_MYSQL=ON \
        	    -DDISABLE_PQ=ON -DDISABLE_ODBC=ON
        	$ make
        	$ sudo make install
        	$ popd
        	```

	3. KENNEL のビルド

		最初に `pushd` するときの `wandbox` ディレクトリは CATTLESHED で clone した `wandbox` ディレクトリ

        	```
        	$ pushd wandbox/kennel2
        	$ ./autogen.sh
        	$ ./configure --prefix=/opt/wandbox --with-cppcms=/opt/cppcms --with-cppdb=/opt/cppdb
        	$ make
        	$ sudo make install
        	$ popd
        	```

	4. ちょっと調整

		```
		$ sudo chown -R wandbox:wandbox /opt/wandbox
                $ sudo echo 'Defaults:root !requiretty' >> /etc/sudoers
                $ sudo vi /opt/wandbox/etc/kennel.json
			^ domain とか変更する
		```

1. Systemd の設定
	
	* CATTLESHED

		1. `systemd/wandbox-cattleshed.service` ファイルを `/usr/lib/systemd/system/` にコピー
		2. `# systemd enable wandbox-cattleshed.service`
		3. `# systemd start wandbox-cattleshed.service`
		4. `ps` とかすると cattleshed が動いているはず

	* KENNEL
		
		1. `systemd/wandbox-kennel.service` ファイルを `/usr/lib/systemd/system/` にコピー
                2. `# systemd enable wandbox-cattleshed.service`
                3. `# systemd start wandbox-cattleshed.service`
                4. `ps` とかすると kennel が動いているはず

	この時点で telnet とかで 127.0.0.1:3500 に `GET / HTTP/1.0` とか投げるとなんか返ってくるはず

1. リバースプロキシの作成または設定

	フロントの 80 なり 443 に HTTP フロントエンドを設けて 127.0.0.1:3500 へ転送するようにする

	起動してアクセスすると何か見えるはず

1. php-build のインストール

	```
	$ git clone https://github.com/php-build/php-build.git
        $ sudo php-build/install.sh
	```

1. php-build の設定

	* `/usr/local/share/php-build/default_configure_options` に `--enable-intl` を追加
	* `./php-build/php_dom.patch` を `/usr/local/share/php-build/patches/` にコピー

1. 各バージョンの PHP のインストール

	* `$ sudo ./bin/update` でそれなりに
	* 終わったら `$ sudo systemctl restart wandbox-cattleshed` で cattleshed を再起動

TODO
----

* cattleshed の python スクリプト使えよ問題
* HHVM

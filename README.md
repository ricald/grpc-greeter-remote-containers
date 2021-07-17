# grpc-greeter-remote-containers

## 内容

- Microsoft Docs の 「[チュートリアル: ASP.NET Core で gRPC のクライアントとサーバーを作成する](https://docs.microsoft.com/ja-jp/aspnet/core/tutorials/grpc/grpc-start?view=aspnetcore-3.1&tabs=visual-studio-code)」をVisual Studio Code の RemoteContainer で実行するソースコード

## 証明書を作成して信頼する手順

1. manually generate self-signed cert

    - create localhost.conf file with the following content:

        ```
        [req]
        default_bits       = 2048
        default_keyfile    = localhost.key
        distinguished_name = req_distinguished_name
        req_extensions     = req_ext
        x509_extensions    = v3_ca

        [req_distinguished_name]
        commonName                  = Common Name (e.g. server FQDN or YOUR name)
        commonName_default          = localhost
        commonName_max              = 64

        [req_ext]
        subjectAltName = @alt_names

        [v3_ca]
        subjectAltName = @alt_names
        basicConstraints = critical, CA:false
        keyUsage = keyCertSign, cRLSign, digitalSignature,keyEncipherment

        [alt_names]
        DNS.1   = localhost
        DNS.2   = 127.0.0.1
        ```

    - generate cert using ```openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout localhost.key -out localhost.crt -config localhost.conf```

    - convert cert to pfx using ```openssl pkcs12 -export -out localhost.pfx -inkey localhost.key -in localhost.crt```

    - (optionally) verify cert using ```openssl verify -CAfile localhost.crt localhost.crt``` which should yield ```localhost.crt: OK```

    - as it's not trusted yet using ```openssl verify localhost.crt``` should fail with

        ```
        CN = localhost
        error 18 at 0 depth lookup: self signed certificate
        error localhost.crt: verification failed
        ```

1. trust this cert:

    - copy localhost.crt to ```/usr/local/share/ca-certificates```

    - trust the certificate using ```sudo update-ca-certificates```

    - verify if the cert is copied to ```/etc/ssl/certs/localhost.pem``` (extension changes)

    - verifying the cert without the CAfile option should work now

        ```
        $ openssl verify localhost.crt 
        localhost.crt: OK
        ```

1. force your application to use this cert
    - update your appsettings.json with the following settings:
        ```
        "Kestrel": {
            "Certificates": {
                "Default": {
                "Path": "localhost.pfx",
                "Password": ""
                }
            }
        }
        ```

### 実行手順

- サーバを実行
    ```
    dotnet run --project ./GrpcGreeter/GrpcGreeter.csproj
    ```

- クライアントを実行
    ```
    dotnet run --project ./GrpcGreeterClient/GrpcGreeterClient.csproj
    ```

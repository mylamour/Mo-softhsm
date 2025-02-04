

<p align="center">
  <img src="https://user-images.githubusercontent.com/12653147/99629727-75269300-2a73-11eb-8dce-16550c9ee1ce.png"  width=345/>
</p>



# Intro

This is a basic usage for `SoftHsm2` and `pkcs11-proxy`. now we are building a env with docker file. it mainly included five part. 
1. Basic Env, also `SoftHsm2` and `pkcs11-proxy` 
2. Upload PGP Public Key when you try to building this image, it was designed for protected the `pin` vaule and `so-pin` vaule
3. Initilized the `slot` with random `32 bit hex` vaule and encrypt the vaule with `PGP Public Key`
4. Enable `pkcs11-proxy`
5. Web Service which provide ability to encrypt/decrypt payload as a services

![image](https://user-images.githubusercontent.com/12653147/100545213-b9950880-3295-11eb-905f-f0d466cc9f2d.png)


# How to use

1. Genreate PGP key with `gpg --full-generate-key` and export the public with `gpg --export xxxx > PIN_PRO.testing.public.asc` to `softhsm2-proxy/publickeys` folder.   make sure the numbers of gpg keys should equal with config file `.env.dev`. field: `HSM_KEYSHARES`

2. Generate a randome string for `TLS-PSK`,  You can use the following command to do that `echo -e $(openssl rand -hex 32):$(openssl rand -hex 32) > TLS-PSK`. it's only use for `pkcs11-tool` 

3. Build the image with `docker-compose --env-file ./config/.env.dev build`, You can change the configuration in `./config/.env.dev`

4. Run the container `docker run --rm -v $PWD/tokens/user02:/var/tokens -p 5657:5657 -p 8443:8443 mo-softhsm2-proxy:latest`

Notices:
>  Please keep use absolute path when we try to mount the `volum`

Now, you will able to use `softhsm2` with `pkcs11-proxy`, and `pkcs11-proxy` was listening on `0.0.0.0:5657` with `TLS` supported

5. `curl` it

```bash
$ curl --header "Content-Type: application/json"  --request POST --data '{"secret_path":"random_test/rsa", "secret_version":"1"}' http://127.0.0.1:8443/gen/key/rsa
{
  "Info": "RSA Key with random_test/rsa Generated"
}
$ curl --header "Content-Type: application/json"  --request POST --data '{"secret_path":"random_test/rsa", "secret_version":"1"}' http://127.0.0.1:8443/gen/key/rsa
{
  "Info": "Key was exsits"
}
$ curl --header "Content-Type: application/json"  --request POST --data '{"secret_path":"random_test/rsa", "secret_version":"2","plaintext":"you are handsome man" }' http://127.0.0.1:8443/key/encrypt/rsa
{
  "ciphertext": "a92b38d38140bd2dcb6651ee0a7001af7e48506fdecf2b30ad5767bd229456be1a5b60d8095ac3702dfd34ee4bbd46c35449ad006a48bd185e4090cfd683da26cb0e18b4c35e0a0bd0e8f11659fed5c95a120b6b9ea970b480b59cbfcd2a1f1805a652a6e31df9377456253e106656086026c54b0c81460d2990726a612a8511d755db4919bae1a3fe78e2850073f53e81b9b2cb12f16cfbc890dce2e47a3e47f9e0da4c03f337f94d5dad12b5e70cc89458730c57fece59f737a2e6fdc6713571bdcbf0178746fa595aba520f7e9050be8b20bc7ae606d96bbeeedd4039d86f0899e0ac5aa60040c5a527e23a788fcf3cda70c70e6b7043e8beec4d85c297b5"
}

$ curl --header "Content-Type: application/json"  --request POST --data '{"secret_path":"random_test/rsa","secret_version":"2", "ciphertext":"593834fa8daa9a0ae9f87f40554318ed652540e4c73ad4e07bce7c4815daee468e79cb365bcff4284762bd330eedcd0ba2b4f107c30c9f391a95a594072b2bc11e8706cff3d690f1e1cfcd146750ab4d6239991ac2aa1f87367ed903385d3a21bb8fd5de333f84efa5468e45a503bf4a3e813b7704486141a0755d11b1afdb97eebdb5ac35a6301fc773a4c9445287dadeadac416a3cdb6cfd9de7262e6e64ff201a27ef7cf4675171de42e8f4dedc75a276a26515a490ad1709a2f7dae0a5767c18c6f887db0748dbb67dcab2aeb206fe3946edca083f6198c7c1794a48e1c00fb9ad9dd6d98b4cb97569205936fce4978f581d16a1d116a4875aadbaeecafc"}' http://127.0.0.1:8443/key/decrypt/rsa
{
  "plaintext": "you are handsome man"
}

$ curl --header "Content-Type: application/json"  --request POST --data '{"secret_path":"random_test/rsa", "secret_version":"2","data":"you are handsome man" }' http://127.0.0.1:8443/key/sign/message
{"signature":"a2060faf5d2be526e37e9e19ad992db53aee7a3cb2abe52ea65867760d3e1ab952e0f608a521a188e064cc3fd667803b38520be80c445cd36f2f71b153af613ce9ffc70d2404e12d0fbdc5843c8221090894056824f635bfe977980db2c956a6506f1b94aada4a071ead225862a2cdf77f61dc857b4af9d9d0f3c7ba3642db8d3c7a7032eb30b578e0f857aeb700d1087d8ff5c3b94a197b3393501da404d0fe239e9ccbce79cc92d355978f98686ee7d28afe5fbb2631613887fa8df27bf98ebb6b7d4c2f8f7832545575d573963b519d59178f16247579b00e3bb8473747c37036e6f432be05bc4df0036e3166f57ba7b5bf1ec93059d4de550f8a20457fef"}

$ curl -F "data=@test.txt" -F "secret_path=random_test/rsa" -F "secret_version=2" 127.0.0.1:8443/key/sign/file
{"signature":"5c40168d2f7c1b3baaaa5828c45dc406860b6a043c8b3fe43d471e67254bf038affabba32fd27148f7ac8a78c0fa6f3457f027a6a0cbdb3b42c283d30b4041842eece25fc51d15d941f2ec5298e2fc1a1d0bb39e22a90b31c92d3491ac9864f0c633076505a9c9d31917db1a8fb9f81f94bb9348ed28e7940b2ca9350b6a7b5d334dcc5e92f75b45c28cddb526fcae897cc9f9c42eaa0ab4b6d6f3c9fbefb3287bf542114bf388f75b022c072735860be928a3cbb36a224f904148c4dd5d3cd4de9d7796be5b97f15c2e5bc20e9f4491cae9a6cd29671a2ad277cfc097a00de14612055e9b93e01b0865601e475704ec911fe8eb043547c3c848d69a66f77534"}

$ curl --header "Content-Type: application/json"  --request POST --data '{"secret_path":"random_test/rsa", "secret_version":"2","data":"you are handsome man","signature":"a2060faf5d2be526e37e9e19ad992db53aee7a3cb2abe52ea65867760d3e1ab952e0f608a521a188e064cc3fd667803b38520be80c445cd36f2f71b153af613ce9ffc70d2404e12d0fbdc5843c8221090894056824f635bfe977980db2c956a6506f1b94aada4a071ead225862a2cdf77f61dc857b4af9d9d0f3c7ba3642db8d3c7a7032eb30b578e0f857aeb700d1087d8ff5c3b94a197b3393501da404d0fe239e9ccbce79cc92d355978f98686ee7d28afe5fbb2631613887fa8df27bf98ebb6b7d4c2f8f7832545575d573963b519d59178f16247579b00e3bb8473747c37036e6f432be05bc4df0036e3166f57ba7b5bf1ec93059d4de550f8a20457fef" }' http://127.0.0.1:8443/key/verify/message
{"Verify Result":true}

$ curl -F "data=@test.txt" -F "secret_path=random_test/rsa" -F "secret_version=2" -F "signature=5c40168d2f7c1b3baaaa5828c45dc406860b6a043c8b3fe43d471e67254bf038affabba32fd27148f7ac8a78c0fa6f3457f027a6a0cbdb3b42c283d30b4041842eece25fc51d15d941f2ec5298e2fc1a1d0bb39e22a90b31c92d3491ac9864f0c633076505a9c9d31917db1a8fb9f81f94bb9348ed28e7940b2ca9350b6a7b5d334dcc5e92f75b45c28cddb526fcae897cc9f9c42eaa0ab4b6d6f3c9fbefb3287bf542114bf388f75b022c072735860be928a3cbb36a224f904148c4dd5d3cd4de9d7796be5b97f15c2e5bc20e9f4491cae9a6cd29671a2ad277cfc097a00de14612055e9b93e01b0865601e475704ec911fe8eb043547c3c848d69a66f77534" 127.0.0.1:8443/key/verify/file
{"Verify Result":true}


```


# Working with Cli

After we start the container with `docker run --rm -it -v $PWD/tokens/user01:/var/tokens mylamour/more-softhsm2-proxy`, you will see the `gpg` file was placed in `$PWD/tokens/user01`, you will get the slot token after working with `gpg -d pinsecret.gpg`, in this repo, the testing key's password was `test`. it's weakness, **you must setting a strong password for prod env.**

Get token from gpg files 
```bash
find . -type f -name "*.pin.gpg" -exec gpg -d {} \; > /tmp/tokens | secret-share-combine 
```

![image](https://user-images.githubusercontent.com/12653147/100181707-da680180-2f15-11eb-8978-fd768375360f.png)


you can find more details about `secret-share-combine` and `secret-share-split` in `softhsm2-proxy/sss/readme.md`
This is just for test purpose. you should get the shares from each `Key Custodian` , and make sure it was only stored in memory.

as you can see in this picture, i was login into container and use below commands to have a testing:
```bash
root@803b461774cd:/# export PKCS11_PROXY_SOCKET="tls://172.17.0.2:5657"
root@803b461774cd:/# pkcs11-tool --module=/usr/local/lib/libpkcs11-proxy.so -L
AvaiLABEL slots:
Slot 0 (0x9a12019): SoftHSM slot ID 0x9a12019
  token label        : DEMO
  token manufacturer : SoftHSM project
  token model        : SoftHSM v2
  token flags        : login required, rng, token initialized, PIN initialized, other flags=0x20
  hardware version   : 2.5
  firmware version   : 2.5
  serial num         : 0b1650b589a12019
  pin min/max        : 4/255
Slot 1 (0x1): SoftHSM slot ID 0x1
  token state:   uninitialized
root@803b461774cd:/# pkcs11-tool --module=/usr/local/lib/libpkcs11-proxy.so -O -l
Using slot 0 with a present token (0x9a12019)
Logging in to "DEMO".
Please enter User PIN:
```

Notices:
> If you connect the softhsm-proxy from another server, please make sure you installed the `libpkca11-proxy.so` and export the TLS PSK file to env. eg. `export PKCS11_PROXY_SOCKET="tls://172.17.0.2:5657"`

![image](https://user-images.githubusercontent.com/12653147/99552255-c1cc8880-29f7-11eb-8bb3-d44c88926be8.png)


python code testing

![image](https://user-images.githubusercontent.com/12653147/100539438-6c9f3b00-3271-11eb-9153-ee2a92fbf3b4.png)


# Notices

1. exec the pkcs11-proxy in background
`nohup sh -c /usr/local/bin/pkcs11-daemon /usr/lib/softhsm/libsofthsm2.so &`
2. gpg encrypt file without interative.
    `gpg -e -u 'SoftHSMv2_ADMIN' --trust-model always -r PGP_RECIPIENT PIN_SECRET`
3. if softhsm2 was build manuly, you should change the lib path.
4. slot token label was able to changed with `docker run -e TOKENLABEL=""` also you can modified the `Dockerfile`
5. mount voulme,  `- ${PWD}/secrets/:/secrets` is not working, `- ${PWD}/secrets:/secrets` is working
6. docker rm none tags image `docker rmi $(docker images --filter "dangling=true" -q --no-trunc)`
7. list user's public keys `gpg --list-public-keys --batch --with-colons | grep pub | cut -d: -f5`
8. once you created a new slot, you need to reload `pkcs-proxy`, or you will not get the new one.

# Resources
* [Pass variable from docker-compose file to docker file](https://github.com/docker/compose/issues/5600)

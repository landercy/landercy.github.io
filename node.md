# 

```sh
npm install -g yarn --registry=https://registry.npmmirror.com/

```


# TROUBLE SHOOTING

## npm ERR! code UNABLE_TO_GET_ISSUER_CERT_LOCALLY

```sh
npm set strict-ssl false
```

## yarn : yarn.ps1 cannot be loaded because running scripts is disabled on this system

```sh
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```


# NOTES

```sh
yarn create vite calendor
yarn dev
```
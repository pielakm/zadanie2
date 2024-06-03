![image](https://github.com/pielakm/zadanie2/assets/102603389/2a22ceba-4cff-4aa3-b8f4-d363a085d685)


# Programowanie Aplikacji w Chmurze Obliczeniowej
_- Numer zadania: 2
- Imię i nazwisko: Mateusz Pielak
- Grupa dziekańska: IO 6.7
- Prowadzący: Dr inż. Sławomir Przyłucki_


## 1. Logowanie się do github'a 
![logowanie](https://github.com/pielakm/zadanie2/assets/102603389/e7facc61-6d7f-496c-bd4a-726095b16864)
## 2. Inicjalizacja i tworzenie nowego repozytorium na potrzeby zadania
![tworzenie repozytorium](https://github.com/pielakm/zadanie2/assets/102603389/cbfdb0cd-7348-44cb-9628-a7dc4a53df7c)
## 3. Przygotowanie i dodanie plików do repozytorium
![git_add](https://github.com/pielakm/zadanie2/assets/102603389/a9c59c71-fd36-42b0-9fa9-1cb0234100bd)
![git_status](https://github.com/pielakm/zadanie2/assets/102603389/2a4b4d38-1779-48f5-9e97-2c44135fc1e2)
![git_commit_and_push](https://github.com/pielakm/zadanie2/assets/102603389/f84a831d-c821-4139-a4c6-124da62e8316)
## 4. Utworzenie zmiennych oraz zmiennych secret:
![tworzenie_secretow](https://github.com/pielakm/zadanie2/assets/102603389/e964cd42-ca2e-417a-a279-2990ae5540b3)
## 5. Uruchomienie całego łańcucha CI przy obrazie bez stwierdzonych zagrożeń (node:22-alpine)
![uruchomienie_lancucha_ci](https://github.com/pielakm/zadanie2/assets/102603389/3b9064cb-857b-4a55-9a86-0ef43a1413b2)
![uruchomienie_lancucha_ci_II](https://github.com/pielakm/zadanie2/assets/102603389/9974e689-f892-4d9f-8325-7f5443f1011d)
![budowa_ok](https://github.com/pielakm/zadanie2/assets/102603389/1e39ee3a-6603-49c6-a0f2-96bbb39429c2)
![budowa_ok2](https://github.com/pielakm/zadanie2/assets/102603389/1153869b-3959-4043-98a1-02ca2156fdd0)
## 6. Uruchomienie całego łańcucha CI przy obrazie ze stwierdzonymi zagrożeniami (node:10)
![ghrunview_wysokieryzyko](https://github.com/pielakm/zadanie2/assets/102603389/2f9886f1-655b-4e02-9330-16c4d18d7488)
![github_opis_ryzyko](https://github.com/pielakm/zadanie2/assets/102603389/ec01109a-f0bd-453c-8f24-e5e02564ef66)
![github_action_run_ryzyko](https://github.com/pielakm/zadanie2/assets/102603389/ba1d968b-386f-4fdb-b6ce-42739e26d14e)
## 7. Utworzenie buildera zad2builder ,który wykorzystuje sterownik docker-container
![tworzenie_buildera](https://github.com/pielakm/zadanie2/assets/102603389/2b89f304-04db-4295-a7f3-2576b6d6bfc1)
## 8. Logowanie do githuba za pomocą tokenu
![logowanie_github](https://github.com/pielakm/zadanie2/assets/102603389/adf8f917-a9ca-43d3-84c3-508f33d67ce2)
## 9. Dodanie tagu i tworzenie paczki na githubie
![tag_and_push](https://github.com/pielakm/zadanie2/assets/102603389/3eaddf3a-b3b3-4d51-8371-424f4aa0ae81)
## 10. Pobranie i uruchomienie obrazu
![Przechwytywanie](https://github.com/pielakm/zadanie2/assets/102603389/9513e68e-3392-4a08-b69b-0f1a8ecd2e2b)
## 11. Zawartość obrazu w przeglądarce
![dzialajace_localhost](https://github.com/pielakm/zadanie2/assets/102603389/ac4fc4ce-8b46-4b77-b93c-dba89de59457)




## Dockerfile
```

# --------- ETAP 1 ------------------------
    FROM scratch as builder

    ADD alpine-minirootfs-3.19.1-aarch64.tar /
        
    RUN apk update && \
        apk upgrade && \
        apk add --no-cache nodejs=20.12.1-r0 \
        npm=10.2.5-r0 && \
        rm -rf /etc/apk/cache
        
    RUN addgroup -S node && \
        adduser -S node -G node
        
    USER node
        
    WORKDIR /home/node/app
        
    COPY --chown=node:node server.js ./server.js
    COPY --chown=node:node package.json ./package.json
    COPY --chown=node:node docker-entrypoint.sh /usr/src/app/
    RUN chmod +x /usr/src/app/docker-entrypoint.sh  
    ENTRYPOINT ["sh", "/usr/src/app/docker-entrypoint.sh"]
        
    RUN npm install
        
    # --------- ETAP 2 ------------------------
        
    FROM node:alpine

    #wysokie ryzyko
    # FROM node:10
        
    # Ustawienie katalogu roboczego w kontenerze
    WORKDIR /usr/app
        
    # Skopiowanie plików aplikacji do kontenera
    COPY package.json /usr/app/
        
    # Skopiowanie reszty plików aplikacji
    COPY ./ /usr/app
        
    # Zainstalowanie zależności aplikacji
    RUN npm install
        
    # Odsłuch na porcie 3000
    EXPOSE 3000
        
    # Uruchomienie serwera
    CMD ["npm", "start"]
        
    # Metadane o autorze Dockerfile
    LABEL author="Mateusz Pielak"

```

## gha_example.yml

```
name: GHAction example
'on':
  workflow_dispatch: null
  push:
    tags:
      - v*
jobs:
  ci_step:
    name: 'Build, tag and push Docker image to DockerHub'
    runs-on: ubuntu-latest
    steps:
      - name: Check out the source_repo
        uses: actions/checkout@v4
      - name: Docker metadata definitions
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: '${{ vars.DOCKERHUB_USERNAME }}/zadanie2'
          flavor: latest=false
          tags: |
            type=sha,priority=100,prefix=sha-,format=short
            type=semver,priority=200,pattern={{version}}   
      - name: QEMU set-up
        uses: docker/setup-qemu-action@v3
      - name: Buildx set-up
        uses: docker/setup-buildx-action@v3
      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: '${{ vars.DOCKERHUB_USERNAME }}'
          password: '${{ secrets.DOCKERHUB_TOKEN }}'
      - name: Build and push Docker image
        id: docker_build
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          platforms: 'linux/amd64,linux/arm64'
          push: true
          cache-from: |
            type=registry,ref=${{ vars.DOCKERHUB_USERNAME }}/zadanie2:cache 
          cache-to: |
            type=registry,ref=${{ vars.DOCKERHUB_USERNAME }}/zadanie2:cache  
          tags: '${{ steps.meta.outputs.tags }}'
      - name: Docker Scout Scan image for vulnerabilities
        id: docker-scout
        uses: docker/scout-action@v1
        with:
          command: cves
          platform: linux/amd64
          image: '${{ steps.meta.outputs.tags }}'
          only-severities: 'critical,high'
          exit-code: true
      - name: Log in to GitHub Container Registry
        if: success()
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: '${{ github.actor }}'
          password: '${{ secrets.MYGITHUB_TOKEN }}'
      - name: Push Docker image to GitHub Container Registry
        if: success()
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: 
            ${{ steps.meta.outputs.tags }}
          
```

## index.js

```
const http = require('http');
const os = require('node:os');
const url = require('node:url');

// Funkcja do obsługi żądań HTTP
const server = http.createServer((req, res) => {
    // Pobranie adresu IP klienta
    const clientIP = req.socket.remoteAddress;
    // Pobranie daty i czasu w strefie czasowej klienta
    const date = new Date();
    const clientTime = date.toLocaleString('pl-PL', { timeZone: 'Europe/Warsaw' });

    // Logowanie informacji o uruchomieniu serwera
    const author = "Mateusz Pielak"; 
    const port = server.address().port;
    console.log(`Serwer uruchomiony przez: ${author}`);
    console.log(`Data uruchomienia: ${date}`);
    console.log(`Serwer nasłuchuje na porcie: ${port}`);

    // Tworzenie treści strony
    const pageContent = `
        <html>
            <head>
                <title>Informacje o kliencie</title>
            </head>
            <body>
                <h1>Adres IP klienta: ${clientIP}</h1>
                <p>Data i czas w strefie czasowej klienta: ${clientTime}</p>
                <p>Dane o autorze: ${author}</p>
            </body>
        </html>
    `;

    // Wysłanie odpowiedzi do klienta
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.write(pageContent);
    res.end();
});
// Uruchomienie serwera na porcie 3000
server.listen(3000, () => {
    console.log('Serwer uruchomiony na porcie 3000...');
});

```

## package.json


```
{
  "name": "zadanie2",
  "version": "1.0.0",
  "description": "Aplikacja do zadania drugiego",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "^4.19.2"
  }
}

```













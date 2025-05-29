Krok 1

Utworzenie katalogu projektu

PS C:\Users\pasko> cd C:\Users\pasko\Downloads
PS C:\Users\pasko\Downloads> mkdir L10_Pasko
    Directory: C:\Users\pasko\Downloads
Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        29.05.2025     23:09                L10_Pask
                                                  o
PS C:\Users\pasko\Downloads> cd L10_Pasko




Krok 2

Umieszczenie kodu projektu z ostatniego zadania w katalogu aktualnego projektu.

Krok 3

Utworzenie pliku .gitignore oraz zapisanie do niego:
node_modules/
.env

Dzięki temu pliki z folderu node_modules nie trafi do Gita.

Krok 4

Inicjowanie repozytorium Git + commit

PS C:\Users\pasko\Downloads\L10_Pasko> git init
Initialized empty Git repository in C:/Users/pasko/Downloads/L10_Pasko/.git/
PS C:\Users\pasko\Downloads\L10_Pasko> git add .
warning: in the working copy of '.gitignore', LF will be replaced by CRLF the next time Git touches it




PS C:\Users\pasko\Downloads\L10_Pasko> git commit -m "feat: pogoda – zad 1 (public/, server.js, Dockerfile, package.json, package-lock.json)"
[master (root-commit) 59f96fb] feat: pogoda ÔÇô zad 1 (public/, server.js, Dockerfile, package.json, package-lock.json)
 6 files changed, 1037 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 Dockerfile
 create mode 100644 package-lock.json
 create mode 100644 package.json
 create mode 100644 public/index.html
 create mode 100644 server.js

Krok 5 

Repozytorium na Github

PS C:\Users\pasko\Downloads\L10_Pasko> gh repo create paskoi/L10_Pasko --public --source=. --push
✓ Created repository Paskoi/L10_Pasko on github.com
  https://github.com/Paskoi/L10_Pasko
✓ Added remote https://github.com/Paskoi/L10_Pasko.git
Enumerating objects: 9, done.
Counting objects: 100% (9/9), done.
Delta compression using up to 20 threads
Compressing objects: 100% (7/7), done.
Writing objects: 100% (9/9), 11.25 KiB | 5.63 MiB/s, done.
Total 9 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/Paskoi/L10_Pasko.git
 * [new branch]      HEAD -> master
branch 'master' set up to track 'origin/master'.
✓ Pushed commits to https://github.com/Paskoi/L10_Pasko.git
PS C:\Users\pasko\Downloads\L10_Pasko>

Krok 6 

Utworzenie repozytorium na dockerhub oraz wygenerowanie tokenu.

Krok 7

Dodanie na Github nowych secretów zawierających:
1. DOCKERHUB_TOKEN – zawiera wygenerowany token
2. DOCKERHUB_USERNAME – zawiera nazwę konta dockerhub

Krok 8 

Wypchnięcie mastera do repozytorium.

PS C:\Users\pasko\Downloads\L10_Pasko> git push origin main

Krok 9

Utworzenie katalogu workflows oraz w nim pliku ci.yaml

Zawartość pliku yaml:
name: CI Multi-Arch Build & Scan

on:
  workflow_dispatch:
  push:
    tags: ['*']

permissions:
  contents: read
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2

      - name: Set up Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to DockerHub (for cache)
        uses: docker/login-action@v2
        with:
          registry: docker.io
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Log in to GitHub Container Registry (GHCR)
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build local image for Trivy scan (amd64 only)
        uses: docker/build-push-action@v4
        with:
          context: .
          file: Dockerfile
          platforms: linux/amd64
          tags: ghcr.io/paskoi/l10-pasko:trivy-scan
          load: true

      - name: Scan for CVE (fail on CRITICAL)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.io/paskoi/l10-pasko:trivy-scan
          severity: CRITICAL
          exit-code: 1

      - name: Push multi-arch image to GHCR
        if: success()
        uses: docker/build-push-action@v4
        with:
          context: .
          file: Dockerfile
          platforms: linux/amd64,linux/arm64
          cache-from: type=registry,ref=${{ secrets.DOCKERHUB_USERNAME }}/l10_pasko-cache:cache
          cache-to: type=registry,ref=${{ secrets.DOCKERHUB_USERNAME }}/l10_pasko-cache:cache,mode=max
          tags: |
            ghcr.io/paskoi/l10-pasko:latest
            ghcr.io/paskoi/l10-pasko:${{ github.sha }}
          push: true


Usunięte zostały zagrożenia high. Występowało jedno związane z biblioteką cross-spawn której zainstalowana wersja to 7.0.3. Po wielu próbach nie byłem w stanie naprawić tego błędu.

Krok 10

Wykonanie skanu przy pomocy Trivy oraz git push

PS C:\Users\pasko\Downloads\L10_Pasko> git commit -m "Trivy scan"
[main 43fde06] ci: dodaj multi-arch build, cache & Trivy scan
 1 file changed, 70 insertions(+)
 create mode 100644 .github/workflows/ci.yml




PS C:\Users\pasko\Downloads\L10_Pasko> git push
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 20 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (5/5), 1.00 KiB | 1.00 MiB/s, done.
Total 5 (delta 1), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/Paskoi/L10_Pasko.git
   59f96fb..43fde06  main -> main

Krok 11

W zakładce Actions uruchamiamy CI Multi-Arch Build & Scan dla naszego workflow.

Krok 12

Wywołanie przez tag

PS C:\Users\pasko\Downloads\L10_Pasko> git tag v1.0.0
PS C:\Users\pasko\Downloads\L10_Pasko> git push origin v1.0.0
Enumerating objects: 78, done.
Counting objects: 100% (78/78), done.
Delta compression using up to 20 threads
Compressing objects: 100% (58/58), done.
Writing objects: 100% (78/78), 22.74 KiB | 2.53 MiB/s, done.
Total 78 (delta 33), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Resolving deltas: 100% (33/33), done.
To https://github.com/Paskoi/L10_Pasko.git
 * [new tag]         v1.0.0 -> v1.0.0
PS C:\Users\pasko\Downloads\L10_Pasko>


Dzięki platforms: linux/amd64,linux/arm64 projekt spełnia wymagania wsparcia 2 architektur.
Dane cash - typ registry.
DockerHub l10_pasko-cache.
cache mode - max. 

Test cve został wykonany przy użyciu Trivy

Link do repozytorium docker https://hub.docker.com/repository/docker/paskoi/l10_pasko-cache/general

Łańcuch został uruchomiony i działa poprawnie

 

# Docker覚書

## 基本
DockerはLinuxカーネルの特定の機能に深く依存しているため、ネイティブではLinux上でしか動作しない。


## Docker Desktop
Docker公式は、Docker Desktopの利用を推奨している

* Docker Desktop for Windows
* Docker Desktop for Mac
* Docker Desktop for Linux

理由は以下
* プラットフォームの違いに関わらず統一した使い勝手になること
* 仮想化によりコンテナをシステムから隔離でき、セキュリティの観点からも利点があること

## Docker Desktop for Windows

WindowsでDockerを使用するためには、WSL2のような仮想化技術を利用してLinux環境を構築し、その上でDocker Engineを実行する必要がある

WSL2を使ってWindows上にLinux環境を構築し、Linuxカーネルを動作させる

Docker Desktopは、このLinuxカーネル上にDocker Engineをインストールし、コンテナを管理します。

Docker Desktop for Windowsは、WSL2をバックエンドとして利用することで、Windows上でLinuxコンテナを効率的に実行できるようにしています。







lm.add_class
lm.landmark
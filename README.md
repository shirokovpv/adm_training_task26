# adm_training_task26
<h1 align="center">Занятие 26. Vagrant-стенд c LDAP на базе FreeIPA</h1>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Описание задания</span></span></h3>
<p><span style="font-weight: 400;">1) Установить FreeIPA</span></p>
<p><span style="font-weight: 400;">2) Написать Ansible-playbook для конфигурации клиента</span></p>
<h3 class="western"><a name="_heading=h.df570rpzx1qg"></a><span style="font-family: Roboto, serif;"><span style="font-size: small;">Используемые ОС</span></span></h3>
<p style="line-height: 108%; margin-bottom: 0.28cm;" align="justify"><span style="font-family: Roboto, serif;">Хостовая ОС Ubuntu 24.04 Desktop с установленным Vagrant 2.3.5. VirtualBox версия 7.0.26 r168464</span></span></p>
<h3 class="western"><span style="font-family: Roboto, serif;"><span style="font-size: small;">Выполнение</span></span></h3>
<p>******************************</p>
<p>Ввиду невозможности в нынешних реалиях воспользоваться стандартными репозиториями Vagrant (в РФ они сейчас не доступны), предложено обходное решение. Использован репозиторий, развернутый на&nbsp;<a href="https://vagrant.elab.pro/" rel="nofollow">https://vagrant.elab.pro/</a></p>
<p>Так как официальный пакет последней версии Vagrant также не доступен для скачивания, пакет взят оттуда же. Версия 2.3.5.</p>
<img width="411" height="88" alt="image" src="https://github.com/user-attachments/assets/7591c684-f543-4bf6-bfa8-3706a7648c97" />
<p>&nbsp;</p>
<p>Для подключения репозитория надо добавить в Vagrant-файл строку:</p>
<p><code>ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'</code></p>
<p>И изменить box_name и box_version (как в репозитории, если туда зайти).</p>
<img width="246" height="145" alt="image" src="https://github.com/user-attachments/assets/82b82c99-e94c-43af-b112-5acaa47cce17" />
<p>******************************</p>
<p><span style="font-weight: 400;">Создадим в каталоге ~/task26 Vagrantfile, в котором будут указаны параметры наших ВМ:</span></p>
<img width="637" height="734" alt="image" src="https://github.com/user-attachments/assets/1b28817f-3d5e-4398-8303-90797906d51e" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Измененный файл прикладываю сюда. После создания Vagrantfile запустим виртуальные машины командой <code>vagrant up</code>. Будут созданы 3 виртуальных машины с ОС CentOS 8. Каждая ВМ будет иметь по 2ГБ ОЗУ и по одному ядру CPU.</span></span></p>
<img width="1093" height="326" alt="image" src="https://github.com/user-attachments/assets/00cfee6e-1bb6-4517-9ddf-65f4d686c9c2" />
<p>&nbsp;</p>
<p><strong>1) Установка FreeIPA сервера</strong></p>
<p><span style="font-weight: 400;">Для начала нам необходимо настроить FreeIPA-сервер. Подключимся к нему по SSH с помощью команды: <code>vagrant ssh ipa.otus.lan</code> и перейдём в root-пользователя: <code>sudo -i</code></p>
<img width="536" height="76" alt="image" src="https://github.com/user-attachments/assets/0518beff-8bf5-4146-b98a-3e213c0a6f42" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Начнем настройку FreeIPA-сервера:</span></p>
<ul>
<li style="font-weight: 400;"><span style="font-weight: 400;">Установим часовой пояс: <code>timedatectl set-timezone Europe/Moscow</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Установим утилиту chrony: <code>yum install -y chrony</code></li>
<p>*********</p>
<p>Возможно, проблема с этой конкретной версией, установленной из этого репозитория - команда не хотела выполняться.</p>
<p>После небольшого траблшутинга -</p>
<pre>cd /etc/yum.repos.d/<br />sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*<br />sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*<br />yum update -y<br /></pre>
<p>- все пошло как надо</p>
<p>*********</p>
<li style="font-weight: 400;"><span style="font-weight: 400;">Запустим chrony и добавим его в автозагрузку: <code>systemctl enable chronyd --now</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Выключим Firewall: <code>systemctl stop firewalld</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Отключаем автозапуск Firewalld: <code>systemctl disable firewalld</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Остановим Selinux: <code>setenforce 0</code></li>
<img width="732" height="296" alt="image" src="https://github.com/user-attachments/assets/ef4af506-48d4-4811-96ed-3b8d86bcf1e2" />
<li style="font-weight: 400;"><span style="font-weight: 400;">Поменяем в файле <strong>/etc/selinux/config</strong> параметр SELINUX на <strong>disabled</strong>
<code>nano /etc/selinux/config</code></li>
<p>*********</p>
<p>Перед этим установим редактор nano (в этой системе его изначально нет) <code>yum install -y nano</code></p>
<p>*********</p>
<img width="880" height="255" alt="image" src="https://github.com/user-attachments/assets/811b5466-4058-4289-8b44-e7aaac4bf933" />
<li style="font-weight: 400;"><span style="font-weight: 400;">Для дальнейшей настройки FreeIPA нам потребуется, чтобы DNS-сервер хранил запись о нашем LDAP-сервере. В рамках данного задания мы не будем настраивать отдельный DNS-сервер и просто добавим запись в файл /etc/hosts: <code>nano /etc/hosts</code></li>
<img width="880" height="105" alt="image" src="https://github.com/user-attachments/assets/ee99410f-f8d0-482a-b5f4-20a0aca87654" />
<li style="font-weight: 400;"><span style="font-weight: 400;">Установим модуль DL1: <code>yum install -y @idm:DL1</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Установим FreeIPA-сервер: <code>yum install -y ipa-server</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Запустим скрипт установки: <code>ipa-server-install</code></li>
<p><span style="font-weight: 400;">Do you want to configure integrated DNS (BIND)? [no]: </span><strong>no</strong></p>
<p><span style="font-weight: 400;">Server host name [ipa.otus.lan]: </span><strong>&lt;Нажимаем Enter&gt;</strong></p>
<p><span style="font-weight: 400;">Please confirm the domain name [otus.lan]: </span><strong>&lt;Нажимем Enter&gt;</strong></p>
<p><span style="font-weight: 400;">Please provide a realm name [OTUS.LAN]: </span><strong>&lt;Нажимаем Enter&gt;</strong></p>
<p><span style="font-weight: 400;">Directory Manager password: </span><strong>&lt;Указываем пароль Otus1234&gt;</strong></p>
<p><span style="font-weight: 400;">Password (confirm): </span><strong>&lt;Дублируем указанный пароль&gt;</strong></p>
<p><span style="font-weight: 400;">IPA admin password: </span><strong>&lt;Указываем пароль Otus1234&gt;</strong></p>
<p><span style="font-weight: 400;">Password (confirm): </span><strong>&lt;Дублируем указанный пароль&gt;</strong></p>
<p><span style="font-weight: 400;">NetBIOS domain name [OTUS]: </span><strong>&lt;Нажимаем Enter&gt;</strong></p>
<p><span style="font-weight: 400;">Do you want to configure chrony with NTP server or pool address? [no]: </span><strong>no</strong></p>
<p><span style="font-weight: 400;">The IPA Master Server will be configured with:</span></p>
<p><span style="font-weight: 400;">Hostname: &nbsp; &nbsp; &nbsp; ipa.otus.lan</span></p>
<p><span style="font-weight: 400;">IP address(es): 192.168.57.10</span></p>
<p><span style="font-weight: 400;">Domain name:&nbsp; &nbsp; otus.lan</span></p>
<p><span style="font-weight: 400;">Realm name: &nbsp; &nbsp; OTUS.LAN</span></p>
<p><span style="font-weight: 400;">The CA will be configured with:</span></p>
<p><span style="font-weight: 400;">Subject DN: &nbsp; CN=Certificate Authority,O=OTUS.LAN</span></p>
<p><span style="font-weight: 400;">Subject base: O=OTUS.LAN</span></p>
<p><span style="font-weight: 400;">Chaining: &nbsp; &nbsp; self-signed</span></p>
<p><em><span style="font-weight: 400;">Проверяем параметры, если всё устраивает, то нажимаем yes</span></em></p>
<p><span style="font-weight: 400;">Continue to configure the system with these values? [no]: </span><strong>yes</strong></p>
<img width="880" height="486" alt="image" src="https://github.com/user-attachments/assets/d92628d2-e813-47e0-adc7-0b0df8488100" />
</ul>
<p><span style="font-weight: 400;">После успешной установки FreeIPA, проверим, что сервер Kerberos может выдать нам билет:</span></p>
<p><span style="font-weight: 400;"><code>kinit admin</code></span></p>
<p><span style="font-weight: 400;"><code>klist</code></span></p>
<img width="880" height="209" alt="image" src="https://github.com/user-attachments/assets/22790e4d-f00a-46ec-8a0e-61f37ba07366" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Мы можем зайти в Web-интерфейс нашего FreeIPA-сервера, для этого на нашей хостовой машине нужно прописать следующую строку в файле Hosts:
192.168.57.10 ipa.otus.lan</span></p>
<img width="880" height="250" alt="image" src="https://github.com/user-attachments/assets/57b86605-21e8-41b7-873f-5efc8ab3bd63" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">После добавления DNS-записи откроем c нашей хост-машины веб-страницу https://ipa.otus.lan/ipa/ui/</span></p>
<img width="1810" height="862" alt="image" src="https://github.com/user-attachments/assets/918e7a57-30e6-4e7f-b366-25032b06ce4d" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Откроется окно управления FreeIPA-сервером. В имени пользователя укажем admin, в пароле укажем наш IPA admin password и нажмём "Войти".</span></p>
<img width="1845" height="366" alt="image" src="https://github.com/user-attachments/assets/c4ea3b29-76f3-4da8-803f-ed814b7e0e93" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Откроется веб-консоль управления FreeIPA. Данные во FreeIPA можно вносить как через веб-консоль, так и средствами командной строки. На этом установка и настройка FreeIPA-сервера завершена.</span></p>
<p><strong>2) Ansible-playbook для конфигурации клиента</strong></p>
<p>*********</p>
<p>Произведем аналогичный траблшутинг для клиентов, чтобы можно было устанавливать пакеты (ну дистрибутив такой, ничего не поделать).</p>
<p><code>vagrant ssh client1.otus.lan</code></p>
<p><code>sudo -i</code></p>
<pre>cd /etc/yum.repos.d/<br />sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*<br />sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*<br />yum update -y<br /></pre>
<p><code>vagrant ssh client2.otus.lan</code></p>
<p><code>sudo -i</code></p>
<pre>cd /etc/yum.repos.d/<br />sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*<br />sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*<br />yum update -y<br /></pre>
<p>*********</p>
<p><span style="font-weight: 400;">В каталоге ~/task26 создадим файл <strong>hosts</strong> со следующими параметрами:</span></p>
<p><code>[clients]</code></p>
<p><code>client1.otus.lan ansible_host=192.168.57.11 ansible_user=vagrant ansible_ssh_private_key_file=./.vagrant/machines/client1.otus.lan/virtualbox/private_key</code></p>
<p><code>client2.otus.lan ansible_host=192.168.57.12 ansible_user=vagrant ansible_ssh_private_key_file=./.vagrant/machines/client2.otus.lan/virtualbox/private_key</code></p>
<img width="1244" height="98" alt="image" src="https://github.com/user-attachments/assets/5a9f80b7-5cd4-4a05-92e8-fbcf0732e814" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Далее создадим файл <strong>provision.yml</strong>, в котором непосредственно будет выполняться настройка клиентов:</span></p>
<img width="846" height="858" alt="image" src="https://github.com/user-attachments/assets/08231acd-ad5d-4956-8d42-6df18a5e4274" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Template файла /</span><em><span style="font-weight: 400;">etc</span></em><span style="font-weight: 400;">/hosts <strong>hosts.j2</strong> выглядит следующим образом:</span></p>
<p><em><span style="font-weight: 400;">127.0.0.1 &nbsp; localhost localhost.localdomain localhost4 localhost4.localdomain4</span></em></p>
<p><em><span style="font-weight: 400;">::1 &nbsp; &nbsp; &nbsp; &nbsp; localhost localhost.localdomain localhost6 localhost6.localdomain6</span></em></p>
<p><em><span style="font-weight: 400;">192.168.57.10 ipa.otus.lan ipa</span></em></p>
<img width="607" height="94" alt="image" src="https://github.com/user-attachments/assets/9e5d1e84-c69d-493a-9af8-81d1a423f692" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Файлы прикладываю сюда.</span></p>
<p><span style="font-weight: 400;">Запускаем плейбук:</span></p>
<p><span style="font-weight: 400;"><code>ansible-playbook -i hosts provision.yml</code></span></p>
<img width="1100" height="909" alt="image" src="https://github.com/user-attachments/assets/9b3d4e21-1963-44c5-97e7-6736d7261d9d" />
<p>&nbsp;</p>
<p><span style="font-weight: 400;">Настройка прошла успешно. Теперь надо проверить работу LDAP:</span></p>
<ul>
<li style="font-weight: 400;"><span style="font-weight: 400;">Авторизуемся на сервере:</span><span style="font-weight: 400;"> <code>kinit admin</code></span></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Создадим пользователя otus-user с паролем Otus1234:</span></li>
<code>ipa user-add otus-user --first=Otus --last=User --password</code>
<img width="613" height="496" alt="image" src="https://github.com/user-attachments/assets/d3c6ed81-afa7-4d17-b98e-81bf917ab1b5" />
<li style="font-weight: 400;"><span style="font-weight: 400;">На хосте client1 или client2 выполним команду <code>kinit otus-user</code>:</span></li>
<img width="613" height="147" alt="image" src="https://github.com/user-attachments/assets/c30805bc-381c-46ec-ac83-4dc187f86530" />
</ul>
<p><span style="font-weight: 400;">Система запросила у нас пароль и потребовала ввести новый. На этом процесс добавления хостов к FreeIPA-серверу завершен.</span></p>
<p><span style="font-weight: 400;">Задание завершено.</span></p>


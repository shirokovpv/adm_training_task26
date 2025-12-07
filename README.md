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
<li style="font-weight: 400;"><span style="font-weight: 400;">Запустим chrony и добавим его в автозагрузку: <code>systemctl enable chronyd -now</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Выключим Firewall: <code>systemctl stop firewalld</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Отключаем автозапуск Firewalld: <code>systemctl disable firewalld</code></li>
<li style="font-weight: 400;"><span style="font-weight: 400;">Остановим Selinux: <code>setenforce 0</code></li>
</ul>















# Урок 12. Автоматизация администрирования. Ansible-1.
В репозитори находятся файлы: [Playbook](nginx.yml), [Vagrantfile](Vagrantfile).

## 1. Создаем вирт. машину с помощью Vagrantfile;
```
vagrant up
```
## 2. Создаем каталог ansible. Создаем конфиг. файла ansible.cfg, в данном конфигируационном файле прописываем пользователя под которым будем подключаться. Расположение inventory файла;

## 3. Создаем файл inventory, называемый hosts;

## 4. Запускаем playbook;
```
ansible-playbook nginx.yml
```
### 4.1 В блоке vars, прописали нестандартный порт 8080 на котором будет работать nginx;
```
  vars:
          nginx_listen_port: 8080
```

### 4.2 В блоке tasks идет установка Nginx, epel-release. Копирования с заменой конфиг. файла nginx из каталога templates в каталог nginx. Добавление в автозагрузку службу nginx;
```
 tasks:
          - name: Install Nginx package from epol repo
            yum:
                    name: nginx
                    state: latest
            notify:
                    - restart nginx
                    
            tags:
                    - nginx-package
                    - packages

          - name: Install EPEL Repo package from standard repo
            yum:
                    name: epel-release
                    state: present
            tags: 
                    - epel-release

          - name: NGINX | Create NGINX config file from template
            template:
                    src: templates/default.conf.j2
                    dest: /etc/nginx/nginx.conf
            notify:
                    - reload nginx
                    
            tags:
                    - nginx-configuration

          - name: Enabled nginx
            systemd:
                    name: nginx
                    enabled: yes
                    masked: no
```

### 4.3 В блоке handlers, указано если будут изменения в конф. файле nginx или обновление пакета nginx, будет выполнено автоматически взависимости от notify перезапуск службы nginx или "перечитает" конфиг. файла nginx;
```
 handlers:
          - name: restart nginx
            systemd:
                    name: nginx
                    state: restarted
                    enabled: yes

          - name: reload nginx
            systemd:
                    name: nginx
                    state: reloaded
```

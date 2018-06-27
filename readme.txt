Данная роль выполняется на локальном хосте (хост с Ansible).

Для успешной работы роли в рабочей директории (рядом с playbook) должна существовать директория "files", данная директория используется для:

    генерации файла конфигурации
    записи лог файлов работы роли


Алгоритм работы:

    Роль находит в дистрибутиве необходимые файлы для запуска утилиты Flyway
    Генерируется конфигурационный файл из шаблона и указанных параметров в описании хоста (local)
    Запускается Flyway с параметром info
    Запускается Flyway с параметром repair
    Запускается Flyway с параметром migrate
    Архивация всех лог файлов из директории "..../playbook/files" в директорию "..../playbook/" (logs.zip)

 


Конфигурация хоста:

#
#Flyway
#
flyway_db_host: 10.68.198.190
flyway_port: 1521
flyway_db_name: KASCXD01
flyway_password: "{{ vault_flyway_password }}"
# Наборы flyway.
flyway:
  baselineVersion: 00.000.00
  outOfOrder: true
  table: ASCC_SCHEMA_HISTORY
  user: AUTOTEST3
  encoding: UTF-8
  validateOnMigrate: "true"
# Наборы flyway.placeholders
flyway_placeholders:
  ascc_aud_schema: AUTOTEST3AUD
  ascc_atl_schema: AUTOTEST3ATL
  ascc_mon_schema: AUTOTEST3MON
  ascc_table_tablespace: STG
  ascc_index_tablespace: STG_INDX
  ascc_app_schema: AUTOTEST3APP


Применяемый шаблон:

#
flyway.driver=oracle.jdbc.driver.OracleDriver
flyway.url=jdbc:oracle:thin:user/password@{{ flyway_db_host }}:{{ flyway_port }}:{{ flyway_db_name }}
# Обрабатываем наборы flyway
{% for item in flyway %}flyway.{{item}}={{flyway[item]}}
{% endfor %}
# Обрабатываем наборы flyway_placeholders 
{% for item in flyway_placeholders %}flyway.placeholders.{{item}}={{flyway_placeholders[item]}}
{% endfor %}


Подключение роли:

---
- hosts: local
  vars:
   work_dir: "{{ playbook_dir }}"
  tasks:
   - name: Flyway
     include_role:
       name: flyway
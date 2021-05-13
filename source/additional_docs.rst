.. _additional_docs:

***************************
B. Дополнительные материалы
***************************

.. _cx_oracle:

B.1. Установка драйвера cx_Oracle на Debian и пример настройки подключения Smarty к Oracle
==========================================================================================

1. Установить пакет ``oracle-instantclient11.2-basic``.

    При установке на Debian необходимо сконвертировать rpm-пакет, \
    предоставляемый Oracle, в deb-пакет с помощью утилиты ``alien``: ::

        alien oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm
        dpkg -i oracle-instantclient11.2-basic_11.2.0.4.0-2_amd64.deb


2. Установить пакет ``oracle-instantclient11.2-devel``: ::

    alien oracle-instantclient11.2-devel-11.2.0.4.0-1.x86_64.rpm
    dpkg -i oracle-instantclient11.2-devel_11.2.0.4.0-2_amd64.deb

3. Установить пакет ``oracle-instantclient11.2-sqlplus``: ::

    alien oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm
    dpkg -i oracle-instantclient11.2-sqlplus_11.2.0.4.0-2_amd64.deb

4. Установить python-модуль ``cx_Oracle``: ::

    pip install cx_Oracle

5. Установить ``libaio1``: ::

    apt-get install libaio1 libaio-dev

.. _smarty_oracle_connection_settings:

B.1.1. Настройка подключения Smarty к Oracle
--------------------------------------------

В файл конфигурации Smarty в секции настроек подключения к БД необходимо прописать: ::

    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.oracle',
            'NAME': "smarty",
            'USER': "smarty", # Имя схемы/пользователя
            'PASSWORD': "password", # Пароль пользователя
            'HOST': "10.0.0.10", # Адрес сервера Oracle
            'PORT': '1521',
            'OPTIONS': {
                'threaded': True,
                'use_returning_into': False,
            },
            'CONN_MAX_AGE': 600,
        }
    }

.. _smarty_postgre_connection_settings:

B.2. Настройка подключения Smarty к PostgreSQL
----------------------------------------------

В файл конфигурации Smarty в секции настроек подключения к БД необходимо прописать: ::

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'smarty',
        'HOST': 'localhost',
    }
}

Параметр Host является обязательным.
Также для PostgreSQL появляется новая зависимость: psycopg2. Устанавливается через pip: ::

pip install psycopg2-binary


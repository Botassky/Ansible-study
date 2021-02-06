.. _kanri-tasho-host-no-tsuika:

***************************************************
管理対象ホストの追加
***************************************************
説明用に管理対象ホストを追加します。 **従来のホストのディレクトリーとは別のディレクトリーに** :file:`Vagrantfile` を作成し、管理対象ホストをデプロイします。以下は :file:`Vagrantfile` の内容です。

.. literalinclude:: ./Vagrantfile
   :language: none

デプロイ後は「 :ref:`kensho-yo-host-no-sakusei` 」で行ったように作成した管理対象ホストの秘密鍵を Ansible サーバーにコピーしたり、:command:`ssh` コマンドでフィンガープリントを収集します。

追加した管理対象ホスト用のインベントリファイル :file:`hosts2.yml` の内容です。

.. code-block:: none

   ---
   all:
     vars:
       ansible_user: vagrant
     hosts:
       centos7:
         ansible_host: 192.168.1.171
         ansible_ssh_private_key_file: ~/.ssh/centos7_key
       centos8:
         ansible_host: 192.168.1.172
         ansible_ssh_private_key_file: ~/.ssh/centos8_key
       debian10:
         ansible_host: 192.168.1.173
         ansible_ssh_private_key_file: ~/.ssh/debian10_key
         ansible_python_interpreter: /usr/bin/python
       ubuntu18:
         ansible_host: 192.168.1.174
         ansible_ssh_private_key_file: ~/.ssh/ubuntu18_key

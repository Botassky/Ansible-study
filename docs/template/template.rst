.. _template:

**************************************************
テンプレート
**************************************************
Ansible で Jinja2 テンプレートエンジンを使用すると、Ansible サーバーから管理対象ホストに変数を埋め込んだファイルをコピーできます。埋め込まれた変数は管理対象ホストにコピーするときに展開されます。

.. tip::

   テンプレート（template）は、文書などのコンピュータデータを作成する上で雛形となるデータ。
   
   最も抽象的なテンプレートは、レイアウトのみのデータで、テキストを流し込むことでレイアウトつき文書となる。具象的なテンプレートは、それ自体文書であり、数箇所の修正または空白への書き込みで目的の文書となる。このほか、さまざまな段階がある。
   
   テンプレートから文書を作る工程は、手動の場合も自動の場合もある。 

   出典： `ウィキペディア　-　テンプレート  (https://ja.wikipedia.org/wiki/テンプレート) <https://ja.wikipedia.org/wiki/%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88>`_

管理対象ホストにログインしたときのメッセージを表示する :file:`/etc/motd` ファイルに値を設定するケースで説明します。

:file:`/etc/motd` ファイルに何も設定されていないときのログインの状態です。

.. code-block:: none

   [vagrant@ansible ansible-files]$ ssh 192.168.1.161 -l vagrant -i ~/.ssh/node1_key 
   Last login: Tue May  5 21:17:17 2020 from 192.168.1.151
   [vagrant@node1 ~]$ ls -l /etc/motd
   -rw-r--r--. 1 root root 0 Jun  7  2013 /etc/motd
   [vagrant@node1 ~]$ 

:file:`/etc/motd` ファイルに設定する値を motd-facts.j2 ファイル（テンプレートファイル）に記述します。テンプレートファイルの拡張子は :file:`.j2` です。

.. code-block:: none

   Welcome to {{ ansible_facts['hostname'] }}.
   {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}
   deployed on {{ ansible_facts['architecture'] }} architecture.

このテンプレートファイル motd_facts.yml を管理対象ホストにコピーします。コピー時に変数（ :ref:`fact-hensu` ）が具体的な値に置き換わります。

.. code-block:: none

   ---
   - name: Fill motd file with host data
     hosts: node1
   
     tasks:
     - name: Copy the template file "motd-facts.j2" to managed node
       template:
         src: motd-facts.j2
         dest: /etc/motd
         owner: root
         group: root
         mode: 0644
       become: yes

実行ログです。

.. code-block:: none

   [vagrant@ansible ansible-files]$ ansible-playbook -i hosts.yml motd-facts.yml 
   
   PLAY [Fill motd file with host data] ************************************************************************************************************************
   
   TASK [Gathering Facts] **************************************************************************************************************************************
   ok: [node1]
   
   TASK [Copy the template file "motd-facts.j2" to managed node] ***********************************************************************************************
   changed: [node1]
   
   PLAY RECAP **************************************************************************************************************************************************
   node1                      : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   
   [vagrant@ansible ansible-files]$ 

管理対象ホスト node1 にログインし、コピーしたメッセージが表示されるか確認します。

.. code-block:: none

   [vagrant@ansible ansible-files]$ ssh 192.168.1.161 -l vagrant -i ~/.ssh/node1_key 
   Last login: Wed May  6 08:35:51 2020 from 192.168.1.151
   Welcome to node1.
   CentOS 7.8
   deployed on x86_64 architecture.
   [vagrant@node1 ~]$ ls -l /etc/motd
   -rw-r--r--. 1 root root 62 May  6 08:35 /etc/motd
   [vagrant@node1 ~]$ cat /etc/motd
   Welcome to node1.
   CentOS 7.8
   deployed on x86_64 architecture.
   [vagrant@node1 ~]$ 

ログイン時に表示されたメッセージからテンプレートファイル内の :ref:`fact-hensu` を展開しコピーしたことがわかります。

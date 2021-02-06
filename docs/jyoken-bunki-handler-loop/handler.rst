.. _handler:

##################################################
ハンドラー
##################################################
タスクを実行した結果、管理対象ホストの状態が変更されたときだけ追加のタスクを実行したいときがあります。例えば、特定のパッケージをインストールや更新した後に、そのパッケージに関するサービスの再起動が必要になるときです。このようなときにハンドラーを使用します。ハンドラーは ``notify`` ディレクティブと ``handlers`` セクションの 2 つで構成します。

notify ディレクティブ
   - 追加タスクが必要なタスクに設定します。
   - 上記の場合、パッケージをインストール／更新するタスクに設定します。

handlers セクション
   - 追加のタスクを定義します。
   - 上記の場合、サービスを再起動するタスクを記述します。

次のインベントリの web グループに含まれる管理対象ホストに最新版の Apache をインストールします。もしすでに Apache がインストール済みであれば最新版に更新します。インストール／更新後は Apache のサービスを再起動します。

.. code-block:: none

   ---
   all:
     vars:
       ansible_user: vagrant
     hosts:
       node1:
         ansible_host: 192.168.1.161
         ansible_ssh_private_key_file: ~/.ssh/node1_key
     children:
       web:
         hosts:
           node2:
             ansible_host: 192.168.1.162
             ansible_ssh_private_key_file: ~/.ssh/node2_key
           node3:
             ansible_host: 192.168.1.163
             ansible_ssh_private_key_file: ~/.ssh/node3_key

プレイです。 Apache のパッケージをインストール／更新する ``yum`` モジュールを使用したタスクに ``notify`` ディレクティブを指定し、 ``handlers`` セクションにサービスを再起動するタスクを記述します。

.. code-block:: none

   ---
   - name: Apache server installed or updated
     hosts: all
     become: yes
     gather_facts: no
   
     tasks:
     - name: latest version Apache installed
       yum:
         name: httpd
         state: latest
       when: inventory_hostname in groups['web'] 
       notify: httpd service restarted
     - name: Process end message
       debug:
         msg: "{{ inventory_hostname }} is finished."
   
     handlers:
     - name: Apache service restarted
       systemd:
         name: httpd.service
         state: restarted
         enabled: yes
       listen: httpd service restarted

``notify`` を設定したタスクと ``handlers`` セクションのタスクは ``notify`` ディレクティブで設定した文字列と ``handlers`` セクション内のタスクの ``listen`` ディレクティブで設定した文字列が合致するタスクが対応します。

実行ログです。 web グループに含まれる管理対象ホスト node2 と node3 は ``notify`` ディレクティブを設定したタスクの実行結果が changed になり、 ``handlers`` セクション内のタスクも実行しました。 node1 は ``notify`` ディレクティブを設定したタスクの実行をスキップ（ skipping ）したので　``handlers`` セクションのタスクは実行しません。

.. code-block:: none

   [vagrant@ansible ansible-files]$ ansible-playbook -i hosts.yml apache.yml 
   
   PLAY [Apache server installed or updated] *******************************************************************************************************************
   
   TASK [latest version Apache installed] **********************************************************************************************************************
   skipping: [node1]
   changed: [node3]
   changed: [node2]
   
   TASK [Process end message] **********************************************************************************************************************************
   ok: [node1] => {
       "msg": "node1 is finished."
   }
   ok: [node3] => {
       "msg": "node3 is finished."
   }
   ok: [node2] => {
       "msg": "node2 is finished."
   }
   
   RUNNING HANDLER [Apache service restarted] ******************************************************************************************************************
   changed: [node2]
   changed: [node3]
   
   PLAY RECAP **************************************************************************************************************************************************
   node1                      : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
   node2                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   node3                      : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   
   [vagrant@ansible ansible-files]$ 

実行ログからわかるように ``handlers`` セクションのタスクは ``tasks`` セクションの実行が終了した後に実行します。 ``notify`` ディレクティブを設定したタスクが changed になった直後に実行しないため注意が必要です。

.. caution::

   ``tasks`` セクション内のタスクが ``handlers`` セクション内のタスクを何回呼び出しても、 ``handlers`` セクション内のタスクは 1 回だけ実行されます。 

再度プレイを実行したときの実行ログです。 web グループの管理対象ホストの Apache は最新版になっているため実行結果は ok です。実行結果が changed ではないので ``handlers`` セクションのタスクは実行しません。

.. code-block:: none

   [vagrant@ansible ansible-files]$ ansible-playbook -i hosts.yml apache.yml 
   
   PLAY [Apache server installed or updated] *******************************************************************************************************************
   
   TASK [latest version Apache installed] **********************************************************************************************************************
   skipping: [node1]
   ok: [node2]
   ok: [node3]
   
   TASK [Process end message] **********************************************************************************************************************************
   ok: [node3] => {
       "msg": "node3 is finished."
   }
   ok: [node1] => {
       "msg": "node1 is finished."
   }
   ok: [node2] => {
       "msg": "node2 is finished."
   }
   
   PLAY RECAP **************************************************************************************************************************************************
   node1                      : ok=1    changed=0    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
   node2                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   node3                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   
   [vagrant@ansible ansible-files]$ 

.. _central-troubleshooting:

Troubleshooting Central 
=========================

.. _reading-container-logs:

Reading container logs
----------------------

If Central is behaving in an unexpected way, it is often helpful to read the Docker container logs with the `docker logs <https://docs.docker.com/engine/reference/commandline/logs/>`_ command for hints as to what has gone wrong.

The containers that are most likely to be helpful for troubleshooting are the `service` container and the `nginx` container. Use the `--tail` and `--since` options to help filter the logs. For example:

* ``docker logs --tail 100 central_service_1`` to see the last 100 lines of the service logs.
* ``docker logs --since 5m central_nginx_1`` to see the last 5 minutes of the nginx logs.

Other commands that are helpful are:

* ``docker ps`` to see the status of all containers.
* ``docker stats`` to see the CPU and RAM usage of all containers.

.. _troubleshooting-emails:

Users aren't receiving emails
-----------------------------

Central uses email as a way to verify user identity when setting or changing passwords. This helps ensure that only the intended user has access to their Central account.

Email sounds like a simple technology but in practice there are many things that can cause message delivery issues. By default, Central is installed with a mail server which can be used without configuration. However, it will not work in every environment. For example:

* Many cloud providers restrict the usage of simple mail servers such as Central's as a spam-prevention strategy
* You can be assigned an IP address that was previously used for sending spam and is therefore blocked by many mail recipients
* Your domain may not be recognized by mail recipients and therefore messages from it may be discarded or marked as spam

To address delivery issues, consider using a dedicated email service such as `Mailjet <https://www.mailjet.com>`_. Because Central doesn't send very many emails, using such a service will generally be a cost-effective way of ensuring email delivery. Once you have an account set up, you will need to :ref:`configure Central to use it <central-install-digital-ocean-custom-mail>`.

If you want to directly send emails from your Central installation, the `mail-tester <https://www.mail-tester.com/>`_ service can help you identify what barriers to email delivery you might have. Create a Central account with the email address that it provides, retrieve your results, and then delete the user. Typically, the first thing you will need to do is :ref:`configure DKIM <central-install-digital-ocean-dkim>` which will provide email recipients confidence that emails were actually sent by your Central server rather than by a spammer pretending to be your server.

.. _troubleshooting-form-preview-:

Preview could not connect with server
-------------------------------------

You may run into a "Could not connect with Server" error message when previewing forms. This error message is typically because your host machine has not made its DNS servers available to Central.

To resolve this problem, first identify your upstream DNS servers. Run ``cat /run/systemd/resolve/resolv.conf`` to see your current list of nameservers with their IP addresses. They will look like this:

  .. code-block:: console

       nameserver 1.2.3.4
       nameserver 9.8.7.6


Now, run ``nano /etc/docker/daemon.json`` to make those nameservers and, optionally, the Google DNS (8.8.8.8) as a fallback available to Docker. Your ``daemon.json`` file will look like the snippet below, but you will need to replace ``1.2.3.4`` and ``9.8.7.6`` with your nameservers' IP addresses.

  .. code-block:: console

       {
           "dns": ["1.2.3.4", "9.8.7.6", "8.8.8.8"]
       }

Finally, stop the containers, restart Docker, and bring the containers back up with ``docker-compose stop``, ``systemctl restart docker`` and ``docker-compose up -d``.

.. _migration-fails-due-to-out-of-memory-error:

Migration fails due to out-of-memory error
------------------------------------------

During upgrades, some versions of Central may perform complex database migrations that need more memory than the 2GB typically allocated to Central.

If you get an error suggesting that the JavaScript heap is out of memory, try :ref:`increasing allocated memory <central-install-custom-memory>`.

.. _export-produces-corrupt-zip:

Export produces corrupt zip
---------------------------

If you have installed Central on a 1GB server or your forms collect many large media files, you may encounter problems exporting submission .zip files. Usually, the .zip file will end up being empty, or much smaller than expected and possibly corrupt.

If you are expecting to collect media files, we recommend having at least 2GB of memory. When collecting images, we recommend :ref:`specifying a maximum size in form design <scaling-down-images>`.

If you still run into problems, try :ref:`increasing allocated memory <central-install-custom-memory>`.

.. _file-upload-fails-with-413:

File upload fails with 413
---------------------------

If you get an error `413` when trying to upload a submission or when trying to upload a form attachment, the file you are trying to upload is too large. By default, files up to 100MB are accepted. We typically recommend reducing the size of the files to upload if possible. For example, :ref:`images can be scaled down in form design <scaling-down-images>`.

If you absolutely must upload files over 100MB, you can change the `client_max_body_size` `nginx` directive:

  .. code-block:: console

    cd
    cd central
    docker-compose stop
    nano files/nginx/odk.conf.template
    <modify the nginx conf value for client_max_body_size>
    docker-compose up -d

.. _troubleshooting-docker-compose-down:

Database disappeared after running Docker commands
--------------------------------------------------

.. warning::
  If you are experiencing data loss, the most important thing to do first is to stop and think through your next steps (ideally with a colleague, who can review those steps). Rushing to act without a plan will most certainly make the situation worse.

  If you do not have a backup, do not reboot or restart the machine. Instead, take a live, full disk backup of the machine so you have a fallback. If you are using DigitalOcean, see `how to create snapshots <https://docs.digitalocean.com/products/images/snapshots/how-to/snapshot-droplets/>`_.

It is possible to accidentally reset the database by running the down command with docker-compose. We are working on a way to prevent this error in the future. For now, if you have run this command and your data has disappeared, you can follow these steps to relocate the data and attach it back to your server:

1. Run the following command: ``docker inspect --type container central_postgres_1 -f '{{(index .Mounts 0).Source}}'``. It should print out a long name starting with /var/lib/docker/volumes/ and ending in a long string of letters and numbers. Copy those letters and numbers and set them aside. They correspond to the location of your current (reset) database.
2. Run ``docker volume ls``. This will tell you all the locations that docker has stored information. We need to find the location that contains your old data.
3. For each long string of letters and numbers you just printed out, run ``file /var/lib/docker/volumes/{letters and numbers}/_data/pg_hba.conf``. So for example, ``file /var/lib/docker/volumes/cd597c21c7f0920fd46001dfd36d454/_data/pg_hba.conf``.
4. If it tells you ``No such file or directory``, move onto the next row and try again with the ``file`` command.
5. If it says ``ASCII text``, you have found database data. But if the string of letters and numbers you just pasted is the same as what you found in step 1, it's not the data you're looking for. Move onto the next set of letters and numbers and try again with step 3.
6. Hopefully you found the data before you got to the end of the list. We found two sets of important letters and numbers following these steps: one in step 1 and one is step 5. Call these FIRST and SECOND, respectively.
7. Now to restore the data, you'll want to run the following commands:

  .. code-block:: console

    cd
    cd central
    docker-compose stop
    pushd /var/lib/docker/volumes
    mv FIRST postgres.data.bak
    mv SECOND FIRST
    popd
    docker-compose up -d

Go to your site in a browser and try to log in with an account that previously existed. If that doesn't immediately work, try doing another ``docker-compose stop`` followed by ``docker-compose up``.
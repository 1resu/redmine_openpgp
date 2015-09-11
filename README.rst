===============
Redmine OpenPGP
===============

A plugin for Redmine to enhance the security of email communication by

- de-/encrypting in-/outgoing emails with the OpenPGP standard
- filtering content of unencrypted emails

developed for `C3S - (Cultural Commons Collecting Society) <https://c3s.cc>`_


Details
=======

Users may

- *add* / *remove* their public PGP key
- *see* the public PGP key of the redmine server

Administrators may

- choose to *pass* / *reject* incoming mails without a valid signature for all projects
- activate outgoing mail handling for *all* / *selected* / *no* projects
- *add* / *generate* / *remove* a private PGP key for the redmine server (both *server-side* / *client-side*)
- choose to *pass* / *filter* / *block* outgoing unencrypted mails
- add a *footer message* to filtered mails

Encrypted mails may be

- *PGP/MIME* or *PGP/Inline* (incoming)
- *PGP/MIME* (outgoing)

Unencrypted mails may be

- *blocked*: no mail is sent
- *filtered*: body is reduced to the link to the added / updated object and an invitation to add the public PGP key; headers & subject are unchanged
- *unchanged*: mail is sent unchanged

Notifications affected:

- attachments_added
- document_added
- issue_add
- issue_edit
- message_posted
- news_added
- news_comment_added
- wiki_content_added
- wiki_content_updated


Dependencies
============

- gpg (http://www.gnupg.org/download/)
- gpgme (https://github.com/ueno/ruby-gpgme)
- mail-gpg (https://github.com/jkraemer/mail-gpg)


Compatibility
=============

This plugin has been tested with
::

    gnupg    1.4.18
    ruby     2.1.5p273
    rails    4.2.3
    redmine  3.1.0
    gpgme    2.0.9
    mail-gpg 0.2.4

*Note:* ``gpg`` == 2.0.X will not work using passphrases, as pinentry is enforced to run asking for the password (see `here <https://stackoverflow.com/a/27768542>`_). In the redmine logs this results in ``Email delivery error: Bad passphrase`` messages, as pinentry cannot provide the passphrase. ``gpg`` >= 2.1 will probably work, if a gpgme passphrase callback function is added to the code (but is still missing). To resolve this problem, use an empty passphrase, or downgrade to 1.X (see `here <https://github.com/C3S/redmine_openpgp/issues/3#issuecomment-139236414>`_ for debian) or install 1.X parallel, symlink ``/usr/bin/gpg`` to ``/usr/bin/gpg2`` and check ``/usr/bin/gpg2 --version``.


Installation
============

#. Change into plugin directory

     ``$cd /path/to/redmine/plugins``

#. Clone this repo into ``/path/to/redmine/plugins/openpgp``

     ``$git clone https://github.com/C3S/redmine_openpgp openpgp``

#. Import the public PGP key for signature verification

     ``$git show pgp | gpg --import``

#. Verify the signature
    
     ``$git tag --verify 1.0``

#. Change to current release tag

     ``$git checkout tags/1.0``

#. Change into redmine root directory

     ``$cd /path/to/redmine``

#. Install gems

     ``$bundle install``

#. Migrate database

     ``$RAILS_ENV=production bundle exec rake redmine:plugins:migrate``

#. Restart redmine

     ``$sudo service apache2 restart``

#. Log in as the user owning the redmine process (e.g. ``redmine``)

     ``$su redmine``

#. Ensure, that the gpg ring is created

     ``$gpg --list-keys``

#. Ensure, that the gpg ring folder is owned by the user owning the redmine process (e.g. ``redmine``)

     ``$chown redmine ~/.gnupg``


Configuration
=============

Administrators
--------------

#. Configure redmine

   - *Administration / Settings / Email notifications*

     - Emission email address

   - *Administration / Settings / General*

     - Host name and path
     - Protocol

   - *Administration / Settings / Incoming emails*

     - Enable WS for incoming emails
     - API key
     - Exclude attachments by name: ``*.asc, *.pgp, *.gpg``

#. Configure plugin

   - *Administration / Plugins / Openpgp*

#. Add or generate a private PGP key for the redmine server 

   - *either* server-side (secure)
   - *or* client-side (**INSECURE over http**, more or less secure over https)

*Note:* Using a passphrase for the private key may be only marginally useful, if it is only used for the purpose of this plugin alone. To retrieve the private key, one would need the rights of the user owning the redmine process. With this rights the passphrase may be retrieved by accessing the database. Anyway, having access to the database means having access to the already decrypted messages. Nevertheless, a password should not hurt ``gpg`` 1.X users.

*Note:* The remote server needs enough entropy to generate random, secure keys. If the server side generation process does not proceed or the client side connection has a timeout, connect to the remote server and try ``ls -R /`` several times. If you use ``rngd`` for entropy generation, be advised not to use ``/dev/urandom`` as source for important keys.

*Note:* HTML in encrypted emails are compatible with PGP/MIME and may be activated in the plugin settings. However, users may respond using PGP/Inline breaking the HTML. If you run into problems, you might provide your users with instructions on how to use PGP/MIME or deactivate html again.

Adding an existing private PGP key server-side
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Export the private PGP key (ascii armored, from ``-----BEGIN PGP PRIVATE KEY BLOCK-----`` to ``-----END PGP PRIVATE KEY BLOCK-----``) and save it into a file on the server

#. Login as the user owning the redmine process (e.g. ``redmine``), to use the right gpg key ring

     ``$su redmine``

#. Change into redmine root directory

     ``$cd /path/to/redmine``

#. Use a rake task to add the existing key, deleting the old one. Point ``keyfile`` to the absolute path to the key file and choose a ``secret``:

     ``$RAILS_ENV="production" bundle exec rake redmine:update_redmine_pgpkey keyfile="/path/to/key.asc" secret="passphrase"``

Generating a new private PGP key server-side
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Login as the user owning the redmine process (e.g. ``redmine``), to use the right gpg key ring

     ``$su redmine``

#. Change into redmine root directory

     ``$cd /path/to/redmine``

#. Use a rake task to generate the new key, deleting the old one. Choose a ``secret``:

     ``$RAILS_ENV="production" bundle exec rake redmine:generate_redmine_pgpkey secret="passphrase"``

Managing a private PGP keys client-side
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. Log into redmine as administrator

#. Visit http://REDMINE.URL/pgp (or follow the new "PGP" link in the account menue)

#. Follow the instructions (on the right side)

Users
-----

#. Log into redmine

#. Visit http://REDMINE.URL/pgp (or follow the new "PGP" link in the account menue)

#. Add your public PGP key

#. Copy & paste the public PGP key for the redmine server into a local file on your machine

#. Import this file into your local gpg key ring

*Note:* The private PGP key for the redmine server has to be added by an administrator, before the corresponding public PGP key is displayed.


Uninstallation
==============

#. Change into redmine root directory

     ``$cd /path/to/redmine``

#. Downgrade the database

     ``$RAILS_ENV=production bundle exec rake redmine:plugins:migrate NAME=openpgp VERSION=0``

#. Remove the files

     ``$rm -r /path/to/redmine/plugins/openpgp``


Implementation
==============

The table ``pgpkeys`` is added to the redmine database:

- each entry associates a redmine user (``user_id``) with the unique fingerprint of a key (``fpr``). This allows for matching fingerprints instead of email address, thus enabling redmine users to delete/update their keys and use keys, which don't match their email address
- the entry with ``user_id`` 0 is reserved for the private key of the redmine server additionally containing the secret passphrase (``secret``)

The following gems are used:

- ``mail-gpg`` for de-/encryption and signature handling within ``Mail`` / ``ActionMailer``
- ``gpgme`` to interact with ``gpg`` running on the server

Whenever a key is added:

- the key is imported into the ``gpg`` key ring of the system user owning the redmine process
- an entry is added to the table ``pgpkeys``

Whenever a key is removed:

- the corresponding entry in the table ``pgpkeys`` is deleted
- if there are no other references to this key within the table ``pgpkeys``:

  - the key is **removed from the gpg key ring** as well

Whenever a mail is sent:

- if the plugin is enabled globally or on project level:

  - if the recipient owns a key:

    - the mail is encryted for the recipient
    - if the redmine server owns a key:

      - the mail is signed by the redmine user

  - else: the mail is blocked / filtered / passed unchanged, depending on the plugin settings

Whenever a mail is recieved:

- it will be decrypted if encrypted

- depending on the plugin settings it will be rejected if the signature is invalid


Improvements
============

- Add tests
- Add languages
- Add LDAP integration for importing keys
- Add gpgme passphrase callback for ``gpg`` >= 2.1, retaining compatibility to ``gpg`` < 2


Links
=====

- `GPG <http://www.gnupg.org/gph/en/manual/x56.html>`_ (reference)
- `ActionMailer <http://apidock.com/rails/ActionMailer/Base>`_ (reference)
- `mail <http://www.rubydoc.info/gems/mail>`_ (reference)
- `gpgme <http://www.rubydoc.info/gems/gpgme/2.0.9>`_ (reference)
- `mail-gpg <http://www.rubydoc.info/gems/mail-gpg/0.2.4>`_ (reference)
- `PGP/MIME <http://www.ietf.org/rfc/rfc3156.txt>`_ (RFC)
- `PGP Formats <http://binblog.info/2008/03/12/know-your-pgp-implementation/>`_ (explanation)


Contributions
=============

- `Alexander Blum <https://github.com/timegrid>`_


License
=======
::

    Redmine plugin for email encryption with the OpenPGP standard
    Copyright (C) 2015 Alexander Blum <a.blum@free-reality.net>

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.

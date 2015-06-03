# Email done right!

Μετά από ~~αρκετό πρήξιμο~~ αρκετά επιχειρήματα του
[Erethon](http://blog.erethon.com/blog/2015/02/27/my-mail-setup-using-mutt-slash-offlineimap-slash-imapfilters/),
είπα να ξαναδοκιμάσω να χρησιμοποιήσω το mutt και ένα σωρό άλλα εργαλεία για τη
διαχείρηση του καθημερινού ~~spam~~ mail... Ο claws-mail με εξ-υπηρέτησε αρκετά
καλά μέχρι τώρα αλλά ήρθε η ώρα να μπω στο group των l33t h4x0rs. Εντάξει, εδώ
που τα λέμε έπαιξε και ένα ρόλο η γνώμη των
[Hartman](http://greg.kh.usesthis.com/) και
[Zacchiroli](http://stefano.zacchiroli.usesthis.com/) αλλά σίγουρα όχι όσο του
Νιόνιου! :-)

*Υπάρχει* και *πιο απλός* τρόπος να στήσεις τον mutt με πολλούς λογαριασμούς
email (και τα λοιπά καλούδια του!) αλλά, για λόγους που θα εξηγηθούν παρακάτω,
το δικό μου setup περιλαμβάνει:

* [offlineimap](#offlineimap): Για το συγχρονισμό των email μεταξύ διαφορετικών
  λογαριασμών και servers.
* [msmtp](#msmtp): Για να μπορείς να απαντήσεις σε email ακόμη και χωρίς σύνδεση
  στο ίντερνετ.
* [mutt](#mutt): Ως απλό mail client με πολλαπλά account και PGP support.
* [notmuch](#notmuch): Για mail indexing.
* [tor](#tor): Για mail servers που είναι σε Tor Hidden Services (HS), όπως π.χ.
  αυτοί της
  [Riseup](https://help.riseup.net/en/security/network-security/tor#riseups-tor-hidden-services).
* [unbound](#tor): Για να δρομολογούνται τα DNS requests τόσο προς το
  "κανονικό" internet όσο και προς το κρεμμυδο-internet.

Όπως θα δούμε, οι κωδικοί έχουν γραφτεί plain-text μέσα στα configuration files.
Θεωρώ αυτονόητο ότι όλα τα confs κάθονται πάνω σε encrypted filesystem. Για τους
(ακόμη πιο) παρανοϊκούς, υπάρχουν κι άλλοι τρόποι (που περιγράφονται στα διάφορα
links) ώστε να μην μπαίνουν οι κωδικοί μέσα στα αρχεία! Αν και, αν έχει κάποιος
access στο $HOME μάλλον οι κωδικοί του mail είναι μικρό πρόβλημα...


## OfflineIMAP <a id="offlineimap"></a>

Όνομα πακέτου (στο Debian πάντα!): `offlineimap`

Το config είναι πολύ απλό και βρίσκεται `~/.offlineimaprc`. Ένα παράδειγμα
υπάρχει [εδώ](./offlineimaprc). Το παράδειγμα είναι minimal και έχει comments.
Αυτά που χρειάζονται συμπλήρωμα βρίσκονται ανάμεσα σε `<`, `>`.

Αφού φτιαχτεί το config, για να γίνουν sync τα mails και να φτιαχτεί το
directory structure:

> $ offlineimap

Μπορεί να αργήσει λίγο την πρώτη φορά!

### Άλλα χρήσιμα links
* https://wiki.archlinux.org/index.php/OfflineIMAP


## mSMTP <a id="msmtp"></a>

Όνομα πακέτου: `msmtp`

Το configuration είναι το `~/.msmtprc`. Ένα παράδειγμα είναι το
[msmtprc](./msmtprc).

Για να βρούμε το tls_fingerprint τού κάθε SMTP server μπορούμε να τρέξουμε:

> msmtp -a e --serverinfo --tls --tls-certcheck=off

### Άλλα χρήσιμα links
* http://msmtp.sourceforge.net/doc/msmtp.html
* https://wiki.archlinux.org/index.php/Msmtp


## Mutt <a id="mutt"></a>

Ονόματα πακέτων: `mutt`, `mutt-patched` (για το sidebar)

Όλα τα configuration files για το mutt τα 'χω μάσει στο `~/.mutt/`. Ένα
παράδειγμα του βασικού config είναι το [muttrc](./muttrc) (και αυτό μπαίνει μέσα
στον παραπάνω φάκελο).

Το multiple accounts το κάνει "αυτόματα" ο mSMTP, διαβάζοντας το FROM header.
Για περισσότερη άνεση (και σιγουριά) κάνουμε τα headers editable στον editor,
στο `muttrc`:

> set edit_headers = yes # edit headers in $editor

Επιπλέον, μέσα στο βασικό conf πρέπει να δηλώσουμε τους φακέλους των mailboxes
μας. Αυτό γίνεται στη γραμμή:

> source ~/.mutt/mailboxes

και ένα παράδειγμα δήλωσης βρίσκεται στο [mailboxes](./mailboxes). Ίσως φανεί
χρήσιμο για τη δημιουργεία του αρχείου κάτι του στυλ:

> $ find ~/Mail/MyServer -maxdepth 1 -type d -name "*" -printf "\"+%f\" "

Για (καλύτερο) PGP support:

> $ cp /usr/share/doc/mutt/examples/gpg.rc ~/.mutt

και προσέχουμε να υπάρχει στο muttrc:

> source ~/.mutt/gpg.rc

Σε περίπτωση που θέλουμε να συντάσουμε απαντήσεις και όταν δεν είμαστε online,
αλλάζουμε το `sendmail` στο `muttrc` ως εξής:

> set sendmail = "/usr/local/bin/msmtp-enqueue.sh"

Αφού πρώτα φροντίσουμε να αντιγράψουμε τα αντίστοιχα scripts:

> $ cp /usr/share/doc/msmtp/examples/msmtpqueue/*.sh /usr/local/bin

Πλέον **όλα** τα email που γράφουμε στοιβάζονται στην ουρά του mSMTP και θα σταλούν όταν
εκτελέσουμε:

> $ /usr/local/bin/msmtp-runqueue.sh

Μπορούμε να δούμε τα περιεχόμενα της ουράς εκτελώντας:

> $ /usr/local/bin/msmtp-listqueue.sh

### Χρήσιμα links
* http://stevelosh.com/blog/2012/10/the-homely-mutt/
* http://www.8t8.us/mutt/
* http://www.techrepublic.com/blog/it-security/use-gnupg-with-mutt-to-sign-or-encrypt-e-mail/
* https://wiki.archlinux.org/index.php/Mutt


## Notmuch Indexing <a id="notmuch"></a>

Ονόματα πακέτων: `notmuch`, `notmuch-mutt` (το mutt interface για το notmuch)

Φτιάχνουμε ένα config τρέχοντας:

> $ notmuch setup

και καταλήγουμε με ένα αρχείο (`~/.notmuch-config`) περίπου σαν κι
[αυτό](notmuch-config).

Και κάνουμε index τα mails (το πρώτο αργεί!):

> $ notmuch new

Κατεβάζουμε και βάζουμε κάπου στο `PATH` (π.χ. στο `/usr/local/bin` το
[python-ο-scripto](./python-notmuch.py). Στη συνέχεια, βάζουμε τα αντίστοιχα
shortcuts στο `~/.mutt/muttrc`:

```
# Notmuch shortcuts
macro index / "<enter-command>unset wait_key<enter><shell-escape>python-notmuch.py<enter><change-folder-readonly>~/.cache/mutt_results<enter>"
macro index I "<enter-command>unset wait_key<enter><pipe-message>notmuch-mutt thread<enter><change-folder-readonly>~/.cache/notmuch/mutt/results<enter>"
```

Και για να κάνει αυτόματα re-index μετά από κάθε sync, βάζουμε σε κάθε Account
section του `~/.offlineimaprc` (αρκεί να μπει και σε ένα μόνο account, αν το
κάνουμε sync κάθε φορά) το `postsynchook` command:

```
[Account MyServer]
localrepository = MyServerLocal
remoterepository = MyServerRemote
postsynchook = notmuch new
```


## My Onion Mail (Tor + Unbound + Ferm/Iptables) <a id="tor"></a>

Ονόματα πακέτων: `tor`, `unbound`, `ferm`/`iptables`

TODO

```
/etc/torrc
/etc/unbound/unbound.conf
/etc/ferm/ferm.conf ή /etc/rc.local
```

### Links
* https://grepular.com/Transparent_Access_to_Tor_Hidden_Services


## Για προχωρημένους

### Practical Password Management

Διαχείριση κωδικών μέσω του
[Keepassx](https://wiki.archlinux.org/index.php/OfflineIMAP#KeePass_.2F_KeePassX)
(και άλλες ταρζανιές).

### 'Get Things Done' Workflow

Το workflow του Zack με
[mutt + org-mode](https://upsilon.cc/~zack/blog/posts/2010/02/integrating_Mutt_with_Org-mode/)
interaction.

Αυτά ;-)

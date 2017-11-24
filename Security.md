*WARNING:* This may take ages and its usefulness is doubtful.

* [Secure Deletion of Data from Magnetic and Solid-State Memory](https://www.usenix.org/legacy/publications/library/proceedings/sec96/full_papers/gutmann/index.html) if you are really paranoid. On the other hand, in this case this setup is probably nothing for you.
* [Overwriting Hard Drive Data](http://computer-forensics.sans.org/blog/2009/01/15/overwriting-hard-drive-data/) for a more recent analysis on this topic

Make used space on encrypted partitions indistinguishable (if paranoid, do
this with a temporary encrypted partition):

    dd if=/dev/zero of=/dev/mapper/crypt-root bs=4M
    dd if=/dev/zero of=/dev/mapper/crypt-home bs=4M

Delete old data on non-encrypted partitions (need only if drive was used
before):

    dd if=/dev/zero of=/dev/lunix/boot
    dd if=/dev/zero of=/dev/lunix/usr

You may use the following function from another console (Ctrl+Alt+F2) to get
continuous status updates every 10s on the main console:

    dd_info() {
        while pkill -USR1 '^dd$'; do
            sleep 10s
        done
    }

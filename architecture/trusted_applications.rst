.. _trusted_applications:

Trusted Applications
====================
There are two ways to implement Trusted Applications (TAs), Pseudo TAs and user
mode TAs. User mode TAs are full featured Trusted Applications as specified by
the :ref:`globalplatform_api` TEE specifications, these are simply the ones
people are referring to when they are saying "Trusted Applications" and in most
cases this is the preferred type of TA to write and use.

.. _pta:

Pseudo Trusted Applications
^^^^^^^^^^^^^^^^^^^^^^^^^^^
These are implemented directly to the OP-TEE core tree in, e.g.,
``core/arch/arm/pta`` and are built along with and statically built into the
OP-TEE core blob.

The Pseudo Trusted Applications included in OP-TEE already are OP-TEE secure
privileged level services hidden behind a "GlobalPlatform TA Client" API. These
Pseudo TAs are used for various purposes such as specific secure services or
embedded tests services.

Pseudo TAs **do not** benefit from the GlobalPlatform Core Internal API support
specified by the GlobalPlatform TEE specs. These APIs are provided to TAs as a
static library each TA shall link against (the ":ref:`libutee`") and that calls
OP-TEE core service through system calls. As OP-TEE core does not link with
:ref:`libutee`, Pseudo TAs can **only** use the OP-TEE core internal APIs and
routines.

As Pseudo TAs runs at the same privileged execution level as the OP-TEE core
code itself and that might or might not be desirable depending on the use case.

In most cases an unprivileged (user mode) TA is the best choice instead of
adding your code directly to the OP-TEE core. However if you decide your
application is best handled directly in OP-TEE core like this, you can look at
``core/arch/arm/pta/stats.c`` as a template and just add your Pseudo TA based on
that to the ``sub.mk`` in the same directory.

.. _user_mode_ta:

User Mode Trusted Applications
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
User Mode Trusted Applications are loaded (mapped into memory) by OP-TEE core in
the Secure World when something in Rich Execution Environment (REE) wants to
talk to that particular application UUID. They run at a lower CPU privilege
level than OP-TEE core code. In that respect, they are quite similar to regular
applications running in the REE, except that they execute in Secure World.

Trusted Application benefit from the GlobalPlatform :ref:`tee_internal_core_api`
as specified by the GlobalPlatform TEE specifications. There are several types
of user mode TAs, which differ by the way they are stored.

TA locations
^^^^^^^^^^^^
Plain TAs (user mode) can reside and be loaded from various places. There are
three ways currently supported in OP-TEE.

.. _early_ta:

Early TA
~~~~~~~~
The so-called early TAs are virtually identical to the REE FS TAs, but instead
of being loaded from the Normal World file system, they are linked into a
special data section in the TEE core blob. Therefore, they are available even
before ``tee-supplicant`` and the REE's filesystems have come up. Please find
more details in the `early TA commit
<https://github.com/OP-TEE/optee_os/commit/d0c636148b3a>`_.

.. _ree_fs_ta:

REE filesystem TA
~~~~~~~~~~~~~~~~~
They consist of a cleartext signed ELF_ file, named from the UUID of the TA and
the suffix ``.ta``. They are built separately from the OP-TEE core boot-time
blob, although when they are built they use the same build system, and are
signed with the key from the build of the original OP-TEE core blob.

Because the TAs are signed, they are able to be stored in the untrusted REE
filesystem, and ``tee-supplicant`` will take care of passing them to be checked
and loaded by the Secure World OP-TEE core. Note that this type of TA isn't
encrypted. 

.. _secure_storage_ta:

Secure Storage TA
~~~~~~~~~~~~~~~~~
These are stored in secure storage. The meta data is stored in a database of all
installed TAs and the actual binary is stored encrypted and integrity protected
as a separate file in the untrusted REE filesystem (flash). Before these TAs can
be loaded they have to be installed first, this is something that can be done
during initial deployment or at a later stage.

For test purposes the test program xtest can install a TA into secure storage
with the command:

.. code-block:: bash

    $ xtest --install-ta

.. _ELF: https://en.wikipedia.org/wiki/Executable_and_Linkable_Format

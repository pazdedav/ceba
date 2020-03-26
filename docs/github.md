# Working with GitHub

## Commit signature verification

Using GPG or S/MIME, you can sign tags and commits locally. These tags or commits are marked as verified on GitHub so other people can trust that the changes come from a trusted source. If a commit or tag has a GPG or S/MIME signature that is cryptographically verifiable, GitHub marks the commit or tag as verified.

Repository administrators can enforce required commit signing on a branch to block all commits that are not signed and verified.

GitHub will automatically use GPG to sign commits you make using the GitHub web interface, except for when you squash and merge a pull request that you are not the author of.

- **GPG**:
  - GnuPG is a complete and free implementation of the OpenPGP standard as defined by RFC4880 (also known as PGP). GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management system, along with access modules for all kinds of public key directories. GnuPG, also known as GPG, is a command line tool with features for easy integration with other applications. GnuPG also provides support for S/MIME and Secure Shell (ssh). Since its introduction in 1997, GnuPG is Free Software (meaning that it respects your freedom). It can be freely used, modified and distributed under the terms of the GNU General Public License.
  - You can use GPG to sign commits with a GPG key that you generate yourself. GitHub uses OpenPGP libraries to confirm that your locally signed commits and tags are cryptographically verifiable against a public key you have added to your GitHub account.
- **S/MIME**: You can use S/MIME to sign commits with an X.509 key issued by your organization. You don't need to upload your public key to GitHub.

1. Download and install the [GPG](https://www.gnupg.org/download/) command line tools.
2.
# Multi cloud-storage backup experiment
Using duplicity's multi backend setup to join, for example, N Mega accounts and get a great amount of storage space.

To do this you need:
- The  latest version of [Duplicity](http://duplicity.nongnu.org/) backup sofware;
- The [mega.py](https://github.com/richardasaurus/mega.py) Python library for the Mega.co.nz API;
- The answer from David Foerster [here](http://askubuntu.com/questions/459792/mega-object-has-no-attribute-get-name-from-file-when-writing-files-to-mega);
- N Mega accounts;

## Duplicity Setup
You need to setup duplicity for a multi backend configuration.
Create a `/home/you/config.json` file with this content
```
[
  {
    "url": "mega://USERNAME1:PASSWORD1@mega.co.nz"
  },
  {
    "url": "mega://USERNAME2:PASSWORD2@mega.co.nz"
  },
  {
    "url": "mega://USERNAME3:PASSWORD3@mega.co.nz"
  },
  ...
]
```

## mega.py Fix
As you can see from the project repo, it has been declared deprecated. :(
...But it still works with this little trick!

- Edit (as root) this file: `/usr/local/lib/python2.7/dist-packages/mega/mega.py`

- Add to the class `class Mega(object)` this code:
```python
def get_name_from_file(self, file):
  for key, value in file.items():
    if 'a' in value and 'n' in value['a']:
      return value['a']['n']
  raise RequestError("Could not find the file attribute.")
```

## Backup and Resore commands
You can now backup a `/path/to/your/folder` with this command:
```
duplicity /path/to/your/folder multi:///home/you/config.json
```
and restore it to `/path/to/restore` with:

```
duplicity multi:///home/you/config.json /path/to/restore
```

Duplicity will perform the backup using a round robin algorithm to save its backup files in the different cloud storage accounts. Assuming, for example, that your accounts are all free ones, you'll get a total available backup space of Nx50 Gb.

I don't think this can be a truly reliable solution for backups, but I can say that it has been a nice after-dinner experiment! :D

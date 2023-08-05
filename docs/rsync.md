sudo rsync -e "ssh -i remote.key" -chavzP --stats /storage/nfsvol/my_folder admin@1.2.3.4:/storage/data1/nfsvol --rsync-path="sudo rsync"

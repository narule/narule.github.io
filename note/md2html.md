# md2html
## Markdown file monitor  2html



> when blog folder's file have change,update  filename.md > filename.html



### monitor shell

###  

```shell
#!/bin/bash

SRCDIR=/home/data/narule/markdown/

# inotifywait -mr --timefmt '%d/%m/%y %H:%M' --format '%T %w %f %e' -e create,delete,close_write,attrib,moved_to $SRCDIR  >> /var/log/narule_change.log

inotifywait -mqr --timefmt '%d/%m/%y %H:%M' --format '%T %w %f %e' -e 'create,delete,modify' $SRCDIR | while read DATE TIME DIR FILE EVENT; do

FILECHANGE="${DIR}""${FILE}"

# echo $EVENT >> /var/log/narule_change.log
echo "$EVENT"
if [[ $EVENT == "CREATE,ISDIR" ]]
        then
        echo "create folder"
        NEWDIR="${DIR/markdown/html}""${FILE}"
        mkdir -p "${NEWDIR}"
elif [[ $EVENT == "DELETE,ISDIR" ]]
        then
        echo "delete folder"
        NEWDIR="${DIR/markdown/html}""${FILE}"
        echo "${NEWDIR}"
        rm -rf  "${NEWDIR}"
elif  [[ $EVENT == "CREATE" ]]
        then
        echo "create html file"
        NEWDIR="${DIR/markdown/html}"
        NEWFILE="${FILE/%.md/.html}"
        markdown_py -o html4 "${DIR}""${FILE}" > "${NEWDIR}${NEWFILE}"
        echo "${FILE}" >> "${NEWDIR}""${NEWFILE}"
        echo "At ${TIME} on ${DATE}, file "${NEWDIR}${NEWFILE}" was creat" >> /var/log/narule_change.log

#$FILECHANGE=${DIR}${FILE}
elif [[ $EVENT == "MODIFY" ]]
        then
        NEWDIR="${DIR/markdown/html}"
        NEWFILE="${FILE/%.md/.html}"
        markdown_py -o html4 "${DIR}""${FILE}" > "${NEWDIR}${NEWFILE}"
        echo "At ${TIME} on ${DATE}, file "${NEWDIR}${NEWFILE}" was modify" >> /var/log/narule_change.log
elif
[[ $EVENT == "DELETE" ]]
        then
        NEWDIR="${DIR/markdown/html}"
        NEWFILE="${FILE/%.md/.html}"
        rm -rf "${NEWDIR}${NEWFILE}"
        echo "At ${TIME} on ${DATE}, file "${NEWDIR}${NEWFILE}" was delete" >> /var/log/narule_change.log

fi
#rsync -avze 'ssh' $SRCDIR root@${DESTHOST}:${DESTHOSTDIR} &>/dev/null && \
#echo "At ${TIME} on ${DATE}, file $FILECHANGE was backed up via rsync" >> /var/log/filesync.log



done

```




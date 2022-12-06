# Практика HDFS

## Задания
1. Когда мы перетащили файлы с произведением Льва Толстого – мы перетащили их в файловую систему виртуальной машины, но не в HDFS, соответственно, в первую очередь нам нужно перенести их в папку нашего пользователя именно на HDFS.
2. После того, как файлы окажутся на HDFS попробуйте выполнить команду, которая выводит содержимое папки. Особенно обратите внимание на права доступа к вашим файлам.
3. Далее сожмите все 4 тома в 1 файл.
4. Теперь давайте изменим права доступа к нашему файлу. Чтобы с нашим файлом могли взаимодействовать коллеги, установите режим доступа, который дает полный доступ для владельца файла, а для сторонних пользователей возможность читать и выполнять.
5. Попробуйте заново использовать команду для вывода содержимого папки и обратите внимание как изменились права доступа к файлу.
6. Теперь попробуем вывести на экран информацию о том, сколько места на диске занимает наш файл. Желательно, чтобы размер файла был удобночитаемым.
7. На экране вы можете заметить 2 числа. Первое число – это фактический размер файла, а второе – это занимаемое файлом место на диске с учетом репликации. По умолчанию в данной версии HDFS эти числа будут одинаковы – это означает, что никакой репликации нет – нас это не очень устраивает, мы хотели бы, чтобы у наших файлов существовали резервные копии, поэтому напишите команду, которая изменит фактор репликации на 2.
8. Повторите команду, которая выводит информацию о том, какое место на диске занимает файл и убедитесь, что изменения произошли.
9. Напишите команду, которая подсчитывает количество строк в вашем файле
10. В качестве результатов вашей работы, запишите ваши команды и вывод этих команд в отдельный файл и выложите его на github.

## Решение
### Команды:

#### 1. Когда мы перетащили файлы с произведением Льва Толстого – мы перетащили их в файловую систему виртуальной машины, но не в HDFS, соответственно, в первую очередь нам нужно перенести их в папку нашего пользователя именно на HDFS.
`docker ps`

`docker cp txts/voyna-i-mir-tom-1.txt 09c979fcdfa8:/`

`docker cp txts/voyna-i-mir-tom-2.txt 09c979fcdfa8:/`

`docker cp txts/voyna-i-mir-tom-3.txt 09c979fcdfa8:/`

`docker cp txts/voyna-i-mir-tom-4.txt 09c979fcdfa8:/`

`docker exec -it 09c979fcdfa8 bash`

`ls`

`hdfs dfs -moveFromLocal voyna-i-mir-tom-1.txt /user/cloudera`

`hdfs dfs -moveFromLocal voyna-i-mir-tom-2.txt /user/cloudera`

`hdfs dfs -moveFromLocal voyna-i-mir-tom-3.txt /user/cloudera`

`hdfs dfs -moveFromLocal voyna-i-mir-tom-4.txt /user/cloudera`

`hdfs dfs -ls /user/cloudera`

`exit`
#### 2. После того, как файлы окажутся на HDFS попробуйте выполнить команду, которая выводит содержимое папки. Особенно обратите внимание на права доступа к вашим файлам.
`docker exec -it 09c979fcdfa8 bash`

`hdfs dfs -ls /user/cloudera`

`exit`
#### 3. Далее сожмите все 4 тома в 1 файл.
`docker exec -it 09c979fcdfa8 bash`

`hadoop archive -archiveName vim.har -p /user/cloudera /user/cloudera`

`hdfs dfs -ls /user/cloudera`

`exit`

На выходе получили `vim.har` с 0 размером:

```
drwxr-xr-x   - root cloudera          0 2022-12-06 11:16 /user/cloudera/vim.har
```

Не файл получился...

Видимо под "сжатием" подразумевалось объединение, поэтому воспользуемся командой `getmerge`. 
Нашел 2 варианта объединения, но оба объединенных файла создались в корне контейнера, а не в hdfs.

`hadoop fs -getmerge -nl /user/cloudera/voyna-i-mir-tom-1.txt /user/cloudera/voyna-i-mir-tom-2.txt /user/cloudera/voyna-i-mir-tom-3.txt /user/cloudera/voyna-i-mir-tom-4.txt /vim_all.txt`

`hdfs dfs -getmerge -nl /user/cloudera/voyna-i-mir-tom-1.txt /user/cloudera/voyna-i-mir-tom-2.txt /user/cloudera/voyna-i-mir-tom-3.txt /user/cloudera/voyna-i-mir-tom-4.txt /voyna-i-mir.txt`

Перенесем один из них в `/user/cloudera/`

`hdfs dfs -moveFromLocal voyna-i-mir.txt /user/cloudera`

`hdfs dfs -ls /user/cloudera`

`exit`
#### 4. Теперь давайте изменим права доступа к нашему файлу. Чтобы с нашим файлом могли взаимодействовать коллеги, установите режим доступа, который дает полный доступ для владельца файла, а для сторонних пользователей возможность читать и выполнять.
`docker exec -it 09c979fcdfa8 bash`

`hdfs dfs -chmod 755 /user/cloudera/voyna-i-mir.txt`

`hdfs dfs -ls /user/cloudera`

`exit`
#### 5. Попробуйте заново использовать команду для вывода содержимого папки и обратите внимание как изменились права доступа к файлу.
`docker exec -it 09c979fcdfa8 bash`

`hdfs dfs -ls /user/cloudera`

`exit`
#### 6. Теперь попробуем вывести на экран информацию о том, сколько места на диске занимает наш файл. Желательно, чтобы размер файла был удобночитаемым.
`docker exec -it 09c979fcdfa8 bash`

`hdfs dfs -du -h /user/cloudera/voyna-i-mir.txt`

`exit`
#### 7. На экране вы можете заметить 2 числа. Первое число – это фактический размер файла, а второе – это занимаемое файлом место на диске с учетом репликации. По умолчанию в данной версии HDFS эти числа будут одинаковы – это означает, что никакой репликации нет – нас это не очень устраивает, мы хотели бы, чтобы у наших файлов существовали резервные копии, поэтому напишите команду, которая изменит фактор репликации на 2.
`docker exec -it 09c979fcdfa8 bash`

В задании написано, что я должен был увидеть 2 числа: фактический размер и место на диске, но я вижу следующее:

`hdfs dfs -du -h /user/cloudera/voyna-i-mir.txt`

```
2.9 M  /user/cloudera/voyna-i-mir.txt
```

Пробуем посмотреть на всю папку:

`hdfs dfs -du -h /user/cloudera`

```
2.9 M    /user/cloudera/vim.har              
719.3 K  /user/cloudera/voyna-i-mir-tom-1.txt
752.3 K  /user/cloudera/voyna-i-mir-tom-2.txt
823.4 K  /user/cloudera/voyna-i-mir-tom-3.txt
681.6 K  /user/cloudera/voyna-i-mir-tom-4.txt
2.9 M    /user/cloudera/voyna-i-mir.txt
```

**Где 2 числа???**

Пока же изменим фактор репликации на 2:

`hdfs dfs -setrep 2 /user/cloudera/voyna-i-mir.txt`

`hdfs dfs -du -h /user/cloudera`

На выходе получаем:
```
2.9 M    /user/cloudera/vim.har
719.3 K  /user/cloudera/voyna-i-mir-tom-1.txt
752.3 K  /user/cloudera/voyna-i-mir-tom-2.txt
823.4 K  /user/cloudera/voyna-i-mir-tom-3.txt
681.6 K  /user/cloudera/voyna-i-mir-tom-4.txt
2.9 M    /user/cloudera/voyna-i-mir.txt
```

Посмотрим, что показывает старый добрый `ls`:

`hdfs dfs -ls /user/cloudera`

```
drwxr-xr-x   - root cloudera          0 2022-12-06 11:16 /user/cloudera/vim.har
-rw-r--r--   3 root cloudera     736519 2022-12-06 10:25 /user/cloudera/voyna-i-mir-tom-1.txt
-rw-r--r--   3 root cloudera     770324 2022-12-06 10:25 /user/cloudera/voyna-i-mir-tom-2.txt
-rw-r--r--   3 root cloudera     843205 2022-12-06 10:38 /user/cloudera/voyna-i-mir-tom-3.txt
-rw-r--r--   3 root cloudera     697960 2022-12-06 10:38 /user/cloudera/voyna-i-mir-tom-4.txt
-rwxr-xr-x   2 root cloudera    3048012 2022-12-06 12:30 /user/cloudera/voyna-i-mir.txt
```

Попробуем изменить еще раз фактор репликации, теперь уже на 4:

`hdfs dfs -setrep 4 /user/cloudera/voyna-i-mir.txt`

`hdfs dfs -du -h /user/cloudera`

```
2.9 M    /user/cloudera/vim.har
719.3 K  /user/cloudera/voyna-i-mir-tom-1.txt
752.3 K  /user/cloudera/voyna-i-mir-tom-2.txt
823.4 K  /user/cloudera/voyna-i-mir-tom-3.txt
681.6 K  /user/cloudera/voyna-i-mir-tom-4.txt
2.9 M    /user/cloudera/voyna-i-mir.txt
```

`hdfs dfs -ls /user/cloudera`

Ничего не изменилось...

`exit`
#### 8. Повторите команду, которая выводит информацию о том, какое место на диске занимает файл и убедитесь, что изменения произошли.
`docker exec -it 09c979fcdfa8 bash`

`hdfs dfs -ls /user/cloudera`

Как мы видели, изменения произошли только в указании фактора репликации.

`exit`
#### 9. Напишите команду, которая подсчитывает количество строк в вашем файле
`docker exec -it 09c979fcdfa8 bash`

`hdfs dfs -cat /user/cloudera/voyna-i-mir.txt | wc -l`

```
10276
```

`exit`
#### 10. В качестве результатов вашей работы, запишите ваши команды и вывод этих команд в отдельный файл и выложите его на github.

Ссылка на эту работу: [https://github.com/MENERGO/DE_Hadoop_HDFS](https://github.com/MENERGO/DE_Hadoop_HDFS)

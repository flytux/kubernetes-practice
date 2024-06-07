### 1. 이미지 목록 확인

```
nerctl images --format "{{.Repositorty}}:{{.Tag}}" > images.txt
```

### 2. Tag가 없는 이미지는 확인하여 Tag 작성


### 3. 비어있는 이미지 가져오기

```
cat images.txt | while read line
do
nerdctl pull ${line}
done
```

### 4. 이미지 tar로 저장

```
nerdctl images | grep -v REPOSITORY | grep -v none | while read line
do
  filename=$( echo "$line" | awk '{print $1":"$2".tar"}' | sed 's|:|@|g ; s|/|+|g' )
  option=$( echo "$line" | awk '{print $1":"$2}' )
  echo "nerdctl save ${option} -o ${filename}"
  nerdctl save "${option}" -o "${filename}"
done
```

### 5. 이미지 로드

```
ls *.tar | while read line
do
   filename=$( echo "$line" )
   echo "nerdctl load -i ${filename}"
   nerdctl load -i "${filename}"
done
```

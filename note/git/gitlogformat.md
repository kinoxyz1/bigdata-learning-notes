
```bash
#!/bin/bash


echo """可输出commit的选项:
  [ 1:ds]
  [ 2:taier]
  [ 3:openmetadata]
  [ 4:idas]
  [ 5:idas-web]
  [ 6:dataservice]
  [ 7:gateway]
  [ 8:datasync]
  [ 9:general]
  [10:store]
  [11:ireport]
  [12:myshop]
  [13:tms]
  [14:tmsweb]
  [15:protal]
  [99:all]"""
read -p "请选择(可多选,用空格隔开): " num
read -p "从这个日期开始(yyyy-MM-dd):" after_time
echo "num: $num, after_time: $after_time" 

now_time=$(date +%Y-%m-%d)
log_file="/Users/kino/Downloads/git_log_$now_time.md"
rm -f $log_file

function get_gitlog()
{
  link="https://kino.cn/$1/"
  branch=$2
  server=${1#*/}
  # git pull origin $branch
  echo "\n### $server" >> $log_file
  # echo "git log $branch --date=iso --pretty=format:\"- [%s] @%an in [%h]($link-/commit/%h)\" --after=\"$after_time\" | grep -v \"Merge\" >> $log_file"
  # $(git log $branch --date=iso --pretty=format:"- [%s] **@%an** in [%h]($link-/commit/%h)" --after="$after_time" | grep -v "Merge" >> $log_file)
  $(git log $branch --date=iso --pretty=format:"- [%s] **@%an** in [%h]($link-/commit/%h)" | tail -n 10 | grep -v "Merge" >> $log_file)
}

function operate()
{
  case $1 in 
  '1')
      cd /Users/kino/works/jzdata/datacenter/dolphinscheduler
      get_gitlog "dmp/dolphinscheduler" "main"
  ;;
  '2')
      cd /Users/kino/works/jzdata/datacenter/taier
      get_gitlog "dmp/taier" "main"
  ;;
  '3')
      cd /Users/kino/works/jzdata/datacenter/openmetadata
      get_gitlog "metadata/openmetadata" "v0.10.0"
  ;;
  '4')
      cd /Users/kino/works/jzdata/datacenter/idas-parent
      get_gitlog "dmp/idas-parent" "main"
  ;;
  '5')
      cd /Users/kino/works/jzdata/datacenter/idas-web
      get_gitlog "dmp/idas-web" "master"
  ;;
  '6')
      cd /Users/kino/works/jzdata/dmp_dataservice
      get_gitlog "jz_dmp/backend/dmp_dataservice" "master"
  ;;
  '7')
      cd /Users/kino/works/jzdata/jz_dm_gateway
      get_gitlog "jz_dmp/backend/jz_dm_gateway" "master"
  ;;
  '8')
      cd /Users/kino/works/jzdata/ninestone/datasync-server
      get_gitlog "ninestone/datasync-server" "dev"
  ;;
  '9')
      cd /Users/kino/works/jzdata/ninestone/jz-general-report
      get_gitlog "ninestone/jz-general-report" "master"
  ;;
  '10')
      cd /Users/kino/works/jzdata/ninestone/jz-store-report
      get_gitlog "ninestone/jz-store-report" "master"
  ;;
  '11')
      cd /Users/kino/works/jzdata/ninestone/ninestone-ireport-parent
      get_gitlog "ninestone/ninestone-ireport-parent" "master"
  ;;
  '12')
      cd /Users/kino/works/jzdata/ninestone/ninestone-ireport-myshop
      get_gitlog "ninestone/ninestone-ireport-myshop" "master"
  ;;
  '13')
      cd /Users/kino/works/jzdata/ninestone/ninestone-tms-parent
      get_gitlog "ninestone/ninestone-tms-parent" "master"
  ;;
  '14')
      cd /Users/kino/works/jzdata/ninestone/ninestone-tms-web
      get_gitlog "ninestone/jz-tms" "master"
  ;;
  '15')
      cd /Users/kino/works/jzdata/ninestone/jz-protal
      get_gitlog "ninestone/jz-protal" "master"
  ;;
  *)
    echo '*'
  esac
}

if [ $num -eq '99' ]; then
  num='1 2 3 4 5 6 7 8 9 10 11 12 13 14 15'
fi
for arr in $num
do 
  operate $arr
done
```
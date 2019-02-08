#!/usr/bin/env bash
set -eu

funccheck() { declare -f "$1" >/dev/null; }
cmdcheck() { type >/dev/null 2>&1 "$@"; }
! cmdcheck git-sub-wget && echo "$0 require git-sub-dir-wget command [umaumax/git\-sub\-wget]( https://github.com/umaumax/git-sub-wget )" && exit 1

# ++++----++++----++++
# { init
# ++++----++++----++++

# [bash - Colorizing your terminal and shell environment? - Unix & Linux Stack Exchange]( https://unix.stackexchange.com/questions/148/colorizing-your-terminal-and-shell-environment )
# [シェルで16x16のカラーパレット表示 - Qiita]( http://qiita.com/ikesato/items/f144c91ed62e7f831edb )

# __※__add echo to '-e' option
COLOR_NC='\033[m' # No Color
COLOR_BLACK='\033[38;05;0m'
COLOR_RED='\033[38;05;1m'
COLOR_GREEN='\033[38;05;2m'
COLOR_YELLOW='\033[38;05;3m'
COLOR_BLUE='\033[38;05;4m'
COLOR_PURPLE='\033[38;05;5m'
COLOR_CYAN='\033[38;05;6m'
COLOR_GRAY='\033[38;05;7m'
COLOR_WHITE='\033[38;05;Fm'

# to enable escape option
echo() { builtin echo -e "$@"; }

# [Bashで文字列をエスケープをする - Qiita]( http://qiita.com/kawaz/items/f8d68f11d31aa3ea3d1c )
sh_escape() {
	local s a=() q="'" qq='"'
	for s in "$@"; do
		a+=("'${s//$q/$q$qq$q$qq$q}'")
	done
	echo "${a[*]}"
}

# [Macの（BSD版）sed での上書き保存 - Qiita]( http://qiita.com/catfist/items/1156ae0c7875f61417ee )
# [Linux（GNU）とMac（BSD）のsedの振る舞いの違いを解決 - Qiita]( http://qiita.com/narumi_888/items/e425f29b84da6b72ad62 )
sedi() { :; }
if sed --version 2>/dev/null | grep -q GNU; then
	sedi() { sed -i "$@"; }
else
	sedi() { sed -i "" "$@"; }
fi

update() {
	url="$1"
	renamed_target_file=""
	[[ $# -ge 2 ]] && renamed_target_file="$2"
	target="${url#https://github.com/}"
	user_repo="${target%%/blob/master/*}"    # file url
	user_repo="${user_repo%%/tree/master/*}" # dir url
	target_file="${target#*/blob/master/}"
	target_file="${target_file#*/tree/master/}"
	raw_url="https://raw.githubusercontent.com/$user_repo/master/$target_file"
	repo_url="https://github.com/$user_repo"

	echo "${COLOR_GREEN}[Checking]${COLOR_NC}: $target"
	# set -e version ([[ *** == ??? ]])はassert代わりのようなものとして利用可能
	# __pipeを使用した時点で全てのプロセスは起動しているので注意(つまり、次のコマンドにEOFを伝えるだけ)__
	master_commit_id=$({
		git ls-remote -h $repo_url
		[[ $? == 0 ]]
	} | grep master | cut -f1)
	# master_commit_id=`{ git ls-remote -h $repo_url; [[ $? != 0 ]] && exit 1; } | grep master | cut -f1`
	[[ $master_commit_id == "" ]] && exit 1

	escapedName=$(sh_escape "$target")
	version_pair="$escapedName $master_commit_id"
	ret=$(cat $log_file_path | grep "$version_pair" | wc -l)
	if ((ret == 0)); then
		echo "${COLOR_BLUE}[UPDATE]${COLOR_NC}: $target"

		# file or dir delete
		# 		rm -rf "$target_file"
		rm -rf "$target_file"
		[[ -n $renamed_target_file ]] && rm -rf "$renamed_target_file"

		# -N: overwrite
		# wget -N $raw_url
		git-sub-wget "$url"

		## download log
		## delete
		sedi ':'"$escapedName"':d' "$log_file_path"
		## add
		echo "$version_pair" >>"$log_file_path"

		[[ -n $renamed_target_file ]] && mv "$target_file" "$renamed_target_file"
		return 0
	else
		echo "${COLOR_GREEN}[NEWEST]${COLOR_NC}: $target is Newest version"
	fi
}

# [実行場所を気にしなくてよいシェルスクリプトの作り方 - Qoosky]( https://www.qoosky.io/techs/927115250f )
# cd `dirname $0`

# ++++----++++----++++
# } init
# ++++----++++----++++

install_file_path="$HOME/.wgit"
log_file_path="$HOME/.config/wgit/home.log"
[[ ! -d "$HOME/.config/wgit" ]] && mkdir -p "$HOME/.config/wgit"
if [[ -f ".wgit" ]]; then
	install_file_path="$PWD/.wgit"
	file_name=$(echo "$install_file_path" | sed -E 's: |/:_:g')
	log_file_path="$HOME/.config/wgit/$file_name.log"
fi

# help
if [[ $# -ge 1 && ("$1" == "-h" || "$1" == "-help" || "$1" == "--help" || "$1" == "help") ]]; then
	echo "install setting file is $install_file_path"
	echo "log file is $log_file_path"
	echo ""
	echo "$0 [github urls ...] :install files"
	echo "$0 clean             :clean log file ($log_file_path)"
	echo "$0 init              :init log and setting files ($install_file_path $log_file_path)"
	echo "$0 remove            :remove log and setting files ($install_file_path $log_file_path)"
	exit 0
fi

# delete log file
if [[ $# -ge 1 && "$1" == "clean" ]]; then
	echo "${COLOR_RED}[CLEAN]${COLOR_NC}: $log_file_path"
	rm $log_file_path
	exit 0
fi

# init
if [[ $# -ge 1 && "$1" == "init" ]]; then
	if [[ -e $install_file_path ]]; then
		echo "${COLOR_RED}[WARNING]${COLOR_NC}: $install_file_path already exists"
		exit 1
	fi
	echo "${COLOR_BLUE}[INIT]${COLOR_NC}: touch $install_file_path && rm $log_file_path"
	touch $install_file_path
	rm $log_file_path
	exit 0
fi

# remove
if [[ $# -ge 1 && "$1" == "remove" ]]; then
	echo "${COLOR_RED}[REMOVE]${COLOR_NC}: rm $install_file_path && rm $log_file_path"
	rm $install_file_path
	rm $log_file_path
	exit 0
fi

lines=""
# NOTE:引数が存在する場合にはインストール設定ファイルを無視する
if [[ $# == 0 ]]; then
	if [[ ! -e $install_file_path ]]; then
		echo "${COLOR_GREEN}[HELP]${COLOR_NC}: To use wgit, please create $install_file_path in your working directory. (use touch $install_file_path or wgit init)"
		exit 1
	fi
	lines=()
	while read line || [ -n "${line}" ]; do
		lines+=("$line")
		# NOTE:catを付加することでgrepした結果の出力がない場合に終了コードが1ではなく，catの終了コードの0となる
	done < <(cat $install_file_path | grep -v "^#" | cat)
else
	lines=("$@")
fi

# create log file
if [[ ! -e $log_file_path ]]; then
	echo "${COLOR_GREEN}[CREATE]${COLOR_NC}: $log_file_path (to save file commit log)"
	touch $log_file_path
fi

# n="${#lines[@]}"
n=$(echo "${lines[@]}" | tr ' ' '\n' | grep '^http\(s\)\?://' | grep -c ^)
if [[ $n == 0 ]]; then
	echo "${COLOR_RED}[WARNING]${COLOR_NC}: no URL infomation from input"
	exit 0
fi

i=1
PWD_=$PWD
for line in "${lines[@]}"; do
	[[ $line == '' ]] && continue
	if [[ $line =~ ^http(s)?:// ]]; then
		echo "${COLOR_GREEN}[$i/$n]:${COLOR_NC} $line"
		update $line
		((i++))
	else
		echo "${COLOR_PURPLE}[CMD]${COLOR_NC}: $line"
		if [[ $line =~ ^cd ]]; then
			cd "$PWD_"
			# 			cd $(sh -c "echo $line")
		fi
		eval $line
	fi
done

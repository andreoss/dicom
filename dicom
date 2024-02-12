#!/bin/bash
# -*- mode: sh -*-
set -e
set -u
set -o pipefail
[ "${DEBUG:-}" ] && set -x
join() {
	local IFS
	IFS="$1"
	shift
	echo "$*"
}

assert_command() {
	local cmd
	cmd="$1"
	shift
	if ! command -v "$cmd" >/dev/null; then
		echo >/dev/stderr "Command '$cmd' not found in \$PATH: $*"
		exit 1
	fi
}
TEMP=$(
	getopt \
		--longoptions overwrite,input:,output:,rate:,width:,height: \
		--options "Oi:f:r:w:h:" -- "$@"
)
eval set -- "$TEMP"
OUTPUT=result.mp4
INPUT=
RATE=12
OVERWRITE=
WIDTH=
HEIGHT=
while true; do
	case "$1" in
	-i | --input)
		INPUT="$2"
		shift 2
		;;
	-o | --output)
		OUTPUT="$2"
		shift 2
		;;
	-r | --rate)
		RATE="$2"
		shift 2
		;;
	-w | --width)
		WIDTH="$2"
		shift 2
		;;
	-h | --height)
		HEIGHT="$2"
		shift 2
		;;
	-O | --overwrite)
		OVERWRITE=1
		shift
		;;
	--)
		shift
		break
		;;
	*) break ;;
	esac
done

assert_command "convert" "'ImagaMagick' is not installed"
assert_command "ffmpeg" "'ffmpeg' is not installed"

if [ -e "$OUTPUT" ]; then
	if [ "$OVERWRITE" ]; then
		rm --force "$OUTPUT"
	else
		echo >/dev/stderr "File already exists: $OUTPUT"
		exit 1
	fi
fi

FILE_LIST=$(mktemp)
trap 'rm --force "$FILE_LIST"' EXIT

# Scan for files
find "$INPUT" -type f | sort >"$FILE_LIST"
echo "Found $(wc -l <"$FILE_LIST") frames"

if [ ! "$WIDTH" ] || [ ! "$HEIGHT" ]; then
	# Calculate width
	SCALE_FILE=$(mktemp)
	trap 'rm --force "$SCALE_FILE"' EXIT
	while read -r DCF; do
		identify -format "%[fx:w] %[fx:h]\n" "$DCF"
	done <"$FILE_LIST" >"$SCALE_FILE"
	WIDTH=$(awk '{print $1}' <"$SCALE_FILE" | sort -rn | head -1)
	HEIGHT=$(awk '{print $2}' <"$SCALE_FILE" | sort -rn | head -1)
	echo "Scale $WIDTH:$HEIGHT"
fi

FFMPEG_OPTS=(
	-loglevel error
	-f image2pipe
	-framerate "$RATE"
	-i -
	-vf "$(
		join , \
			"scale=$WIDTH:$HEIGHT:force_original_aspect_ratio=decrease" \
			"pad=$WIDTH:$HEIGHT:(ow-iw)/2:(oh-ih)/2" \
			"setsar=1"
	)"
	"$OUTPUT"
)
CONVERT_OPTS=()
while read -r DCF; do
	convert "$DCF" "${CONVERT_OPTS[@]}" ppm:'-'
done \
	<"$FILE_LIST" |
	ffmpeg "${FFMPEG_OPTS[@]}"

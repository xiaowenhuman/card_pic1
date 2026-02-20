find . -type f -iname "*.png" ! -iname "*_1.5x.png" -print0 | \
while IFS= read -r -d '' f; do
  if [ ! -f "$f" ]; then
    echo "SKIP(not found): $f"
    continue
  fi
  out="${f%.png}_1.5x.png"
  ffmpeg -hide_banner -loglevel error -y -i "$f" \
    -vf "scale=trunc(iw*1.5):trunc(ih*1.5):flags=lanczos,unsharp=5:5:0.6:3:3:0.0" \
    -pix_fmt rgba "$out" \
  && echo "OK: $f -> $out" \
  || echo "FAIL: $f"
done

上面的命令是把各个图片转化为 *_1.5x.png ,因为原来的图片不能用,太小了，第一步先做个1.5倍的放大。 

下面的命令再加入一个水印。(递归处理所有 PNG（保留目录结构，输出到 wm_out/）)

find . -type f \( -iname "*_1.5x.png" \) -print0 | while IFS= read -r -d '' f; do
  out="../wm_out/${f#./}"
  mkdir -p "$(dirname "$out")"
  WH=$(ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=s=x:p=0 "$f")
  ffmpeg -hide_banner -loglevel error -y -i "$f" -filter_complex \
  "[0:v]format=rgba[base];color=c=black@0.0:s=${WH},format=rgba,drawtext=text='GlobalCollecta':fontcolor=white@0.7:fontsize=w*0.08:x=(w-tw)/2:y=(h-th)/2:shadowcolor=black@0.7:shadowx=3:shadowy=3,rotate=-0.35:ow=rotw(iw):oh=roth(ih):c=none[wm];[base][wm]overlay=(W-w)/2:(H-h)/2" \
  -frames:v 1 -pix_fmt rgba "$out"
  echo "OK: $f -> $out"
done



下面是几个常用的命令： 
1、递归删除名字中带_1.5x.png的文件
find . -type f -name "*_1.5x.png" -exec rm -f {} +


# Environment variable
```
sudoedit /etc/environment

source /etc/environment
```

# chown
```
sudo chown -R testuser:testuser /var/www/test/public_html
```

# service
```
service --status-all
```

# du
Sort directories by size
```
du -sh -- */ | sort -rh
```

# Comm
To find if common between two directory
```
comm -1 -2 <(ls dir1 | sort) <(ls dir2 | sort)
```

# Diff
```
diff /mnt/storage2 /mnt/storage1
diff -r /mnt/storage2 /mnt/storage1
```

# Rename
```
sudo apt-get install rename
```

Usage:
<pre>
rename [options] [Perl regex search/replace expression] [files]
</pre>
From man rename:
<pre>
   -v, --verbose
           Verbose: print names of files successfully renamed.
   -n, --no-act
           No Action: show what files would have been renamed.
</pre>

Example:
```
rename -n 's/(\d)a/$1.1/' *
```
Output
<pre>
Area.v2a.2.h -> Area.v2.1.2.h
Base.v2a.c -> Base.v2.1.c
Slider.v4a.h -> Slider.v4.1.h
</pre>

# mkvtoolnix
```
sudo apt-get install mkvtoolnix
```

Detect track numbers
```
mkvmerge -i some_movie.mkv
```
Sample output:
<pre>
File 'some_movie.mkv': container: Matroska
Track ID 0: video (MPEG-4p10/AVC/h.264)
Track ID 1: audio (AC-3/E-AC-3)
Track ID 2: subtitles (SubRip/SRT)
Chapters: 12 entries
Global tags: 2 entries
</pre>

Extract tracks
Syntax:
```
mkvextract tracks <your_mkv_video> <track_number>:<subtitle_file.srt>
```

# Magick
```
sudo apt install imagemagick-6.q16
```

Usage:
```
convert 'poster.webp' 'poster.png'
```

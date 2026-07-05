---
title: "macOS: Video files in Quarantine"
date: 2017-11-24T15:31:49
feature_image: feature.jpg
summary: 'I am not sure when it exactly started. But some time after I upgraded to High Sierra, VLC started to display a “Verifying the file” prompt with a progress bar whenever I opened a MKV or MP4 file.'
tags:
  - apple
languages:
  - en
---

It did it every time and for every file, which was very annoying — to say the least!

After some research I found that all the files had the special permission `com.apple.quarantine` set, which was only visible via `xattr`.

```shell
$ xattr /path/to/file.mkv
com.apple.quarantine
```

After removing the special quarantine attribute with, the prompt was gone and did not re-appear so far.

```shell
sudo xattr -r -d com.apple.quarantine /path/to/file.mkv
```

I can only speculate that the increased security measures in High Sierra automatically flagged certain files. Why it only affected MKV and MP4 files on my system, I do not know. I think the false-positives were caused by a faulty heuristic, or something similar. But your guess is as good as mine.

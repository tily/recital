# recital

## Usage

```
## download lyrics from genius
$ bundle exec thor download http://genius.com/Nas-ny-state-of-mind-lyrics content/nystateofmind.md

## embed youtube video to page
$ bundle exec thor embed https://www.youtube.com/watch?v=UKjj4hk0pV4 content/nystateofmind.md

## replace `{{<t 00.00>}}` expression with numbers calculated by start = 0.2 sec and BPM = 84.5
$ bundle exec thor annotate 84.5 0.2 content/nystateofmind.md
```

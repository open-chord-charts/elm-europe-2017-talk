[![GitPitch](https://gitpitch.com/assets/badge.svg)](https://gitpitch.com/open-chords-charts/elm-europe-2017-talk/master?grs=github&t=white)

## Dev mode

Launch:

```sh
ln -sf $PWD/../assets/* assets/ ; ln -sf $PWD/../PITCHME.md assets/md/ ; browser-sync start --server --files "**/*" --startPath pitchme.html --serveStatic ../assets/
```
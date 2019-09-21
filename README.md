# jpm
Joke(r) Package Manager

## Install

Paste this macro into your code.

```
(defmacro defnx
  "Defines a new function whose body is downloaded from given URL."
  [name args url]
  (defn parent-dir
    "Returns parent directory of given directory."
    [dir]
    (-> dir joker.filepath/split first joker.filepath/dir))

  (defn mkdirp
    "Creates given directory (and all missing parent directories)."
    [dir mode]
    (if (joker.os/exists? dir)
      nil
      (do (mkdirp (parent-dir dir) mode)
          (joker.os/mkdir dir mode))))
  
  (let [url-hash (-> url joker.crypto/sha256 joker.hex/encode-string)
        home (get (joker.os/env) "HOME")
        jpm-cache-dir (joker.filepath/join home ".jpm" "cache")
        _ (mkdirp jpm-cache-dir 0755)
        source-cache-path (joker.filepath/join jpm-cache-dir url-hash)
        source (try
                 (-> (slurp source-cache-path) read-string)
                 (catch Error e
                   (let [content (-> (joker.http/send {:url url}) :body)]
                     (spit source-cache-path content)
                     (-> content read-string))))
        source-with-name (apply list (-> source (vec) (assoc 1 name)))]
    source-with-name))
```

## Usage

```
(defnx greet [name] "https://gist.githubusercontent.com/maio/8610c52281d169305bf056163b9df21d/raw/5dbf53845cdf3c8455c8f8cdf22028c374cf3c01/greet.clj")

(greet "there")
```
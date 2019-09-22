# jpm
Joke(r) Package Manager

## Install

Paste this macro into your code.

```
(require 'joker.walk)
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

  (defn is-relative-url?
    [url]
    (cond
      (joker.string/starts-with? url ".") true
      true false))

  (defn url-join
    "Join given URLs. a needs to be absolute URL, b may be relative."
    [a b]
    (if (not (is-relative-url? b))
      b
      (let [[aa ab] (joker.string/split a #"//")]
        (str aa "//" (joker.filepath/join (joker.filepath/dir ab) b)))))

  (defn resolve-urls
    "Resolve relative URLs in defnx forms."
    [form parent-url]
    (if (not (list? form))
      form
      (if (= (first form) 'defnx)
        (let [[_ _ _ url] form]
          (if (is-relative-url? url)
            (let [updated (apply list (-> form vec (assoc 3 (url-join parent-url url))))]
              updated)
            form))
        form)))

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
        processed-source (joker.walk/postwalk #(resolve-urls %1 url) (apply list (-> source (vec) (assoc 1 name))))]
    processed-source))
```

## Usage

```
(defnx greet [name] "https://raw.githubusercontent.com/maio/jpm/430ca09e0b12c16a3e666b4d7ad7c9c895913151/test/greet2.joke")

(greet "there")
```
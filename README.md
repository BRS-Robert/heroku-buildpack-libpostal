# heroku-buildpack-libpostal

Installs [libpostal](https://github.com/openvenues/libpostal) (including
training data) into a Heroku slug, enabling the `ruby_postal` gem.

---

## ⚠️ Slug Size Warning

libpostal's training data is **~2 GB uncompressed**. Heroku's slug limit is
**500 MB compressed**. This buildpack will very likely exceed the limit.

If your deploy fails with `Compiled slug size: XXX MB is too large`, switch
to the [microservice approach](https://github.com/openvenues/libpostal) and
run libpostal as a separate containerised Heroku app.

---

## Usage

### 1. Add the buildpack

```bash
# Must come BEFORE heroku-buildpack-ruby
heroku buildpacks:add --index 1 https://github.com/your-org/heroku-buildpack-libpostal
heroku buildpacks:add heroku/ruby
```

Verify order:
```bash
heroku buildpacks
# 1. https://github.com/your-org/heroku-buildpack-libpostal
# 2. heroku/ruby
```

### 2. Trigger detection (choose one)

**Option A** — create a `.libpostal` marker file in your repo root:
```bash
touch .libpostal
git add .libpostal
```

**Option B** — ensure `ruby_postal` is in your `Gemfile` (auto-detected):
```ruby
gem 'ruby_postal'
```

### 3. Deploy

```bash
git push heroku main
```

---

## Configuration

| Env var | Default | Description |
|---|---|---|
| `LIBPOSTAL_VERSION` | `master` | Git branch/tag to clone |

```bash
# Pin to a specific release
heroku config:set LIBPOSTAL_VERSION=v1.1.1
```

---

## Caching

Compiled binaries and training data are cached in Heroku's build cache.
Subsequent deploys restore from cache and skip the ~10 min build + download.

Cache is automatically busted when `LIBPOSTAL_VERSION` changes.

To manually clear the cache:
```bash
heroku builds:cache:purge -a your-app
```

---

## Runtime Environment

The following variables are available inside your dynos:

```
LIBPOSTAL_DATA_DIR  /app/vendor/libpostal/share/libpostal
LD_LIBRARY_PATH     /app/vendor/libpostal/lib
LIBRARY_PATH        /app/vendor/libpostal/lib
PKG_CONFIG_PATH     /app/vendor/libpostal/lib/pkgconfig
C_INCLUDE_PATH      /app/vendor/libpostal/include
PATH                /app/vendor/libpostal/bin:...
```

---

## Buildpack Order

This buildpack **must run before** `heroku/ruby` so that native gem
compilation (e.g. `ruby_postal`) can find the libpostal headers and libraries.

```
1. heroku-buildpack-libpostal   ← installs C library + data
2. heroku/ruby                  ← compiles ruby_postal against it
```

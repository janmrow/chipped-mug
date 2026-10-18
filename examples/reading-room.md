# Reading Room — synthetic example

This synthetic saved-article app is the source of the README example. Saved articles and pins are kept in memory.

Create the four baseline files below in an empty Git repository, commit them on `main`, then create `selected-change` and apply the patch. Ask the agent to explain `main...selected-change`.

## Baseline

### `storage.py`

```python
class ShelfStore:
    def __init__(self):
        self.saved = {}
    def save(self, member_id, article_id):
        self.saved.setdefault(member_id, []).append(article_id)
    def load(self, member_id):
        return list(self.saved.get(member_id, []))
```

### `library.py`

```python
def open_library(store, member_id):
    return [{"article": a} for a in store.load(member_id)]
```

### `api.py`

```python
from library import open_library

def library_response(store, signed_in_member):
    return open_library(store, signed_in_member)
```

### `ui.py`

```python
def labels(rows):
    return [row["article"] for row in rows]
```

## Selected change

Apply this patch and commit the result on `selected-change`:

```diff
diff --git a/api.py b/api.py
index 4f8718b..aa17f37 100644
--- a/api.py
+++ b/api.py
@@ -3,2 +3,5 @@
 def library_response(store, signed_in_member):
     return open_library(store, signed_in_member)
+
+def set_pin(store, signed_in_member, article_id, enabled):
+    store.pin(signed_in_member, article_id, enabled)
diff --git a/library.py b/library.py
index f22b179..77b9a97 100644
--- a/library.py
+++ b/library.py
@@ -1,2 +1,5 @@
 def open_library(store, member_id):
-    return [{"article": a} for a in store.load(member_id)]
+    pins = store.pinned_for(member_id)
+    articles = store.load(member_id)
+    articles.sort(key=lambda a: a not in pins)
+    return [{"article": a, "pinned": a in pins} for a in articles]
diff --git a/storage.py b/storage.py
index e7b9ac2..ef88cbf 100644
--- a/storage.py
+++ b/storage.py
@@ -1,7 +1,13 @@
 class ShelfStore:
     def __init__(self):
         self.saved = {}
+        self.pinned = {}
     def save(self, member_id, article_id):
         self.saved.setdefault(member_id, []).append(article_id)
     def load(self, member_id):
         return list(self.saved.get(member_id, []))
+    def pin(self, member_id, article_id, enabled):
+        pins = self.pinned.setdefault(member_id, set())
+        pins.add(article_id) if enabled else pins.discard(article_id)
+    def pinned_for(self, member_id):
+        return self.pinned.get(member_id, set())
diff --git a/ui.py b/ui.py
index b539070..c263435 100644
--- a/ui.py
+++ b/ui.py
@@ -1,2 +1,2 @@
 def labels(rows):
-    return [row["article"] for row in rows]
+    return [("Pinned: " if row["pinned"] else "") + row["article"] for row in rows]
```

## Recorded explanation

One unedited response recorded using the skill from release v1.0.0:

| Run detail | Value |
| --- | --- |
| Date (UTC) | 2026-10-18 |
| Agent | Codex CLI 0.163.0 |
| Model | `gpt-6.1-sol`, reasoning effort `high` |
| Skill release | `v1.0.0` |
| Skill commit | `367d6a54724772f5f266541ad058cdf151a27af3` |
| Before (`main`) | `154c65f02174f44b2d8bf87173697e5760ac1b57` |
| After (`selected-change`) | `837c4ac9012cdeb596daa015c98a963f563ef72a` |
| Comparison | `main...selected-change` |

The before and after commits were created locally from the baseline and patch above. `SKILL.md` and `LICENSE` from the skill commit were placed in `.agents/skills/chipped-mug/` inside that repository, outside the compared commits. The agent started in a fresh session without the previous explanation in its context.

From the example repository, run:

```sh
codex exec \
  --sandbox read-only \
  --model gpt-6.1-sol \
  -c 'model_reasoning_effort="high"' \
  --json \
  --output-last-message ../response.md \
  'Use $chipped-mug to explain main...selected-change.' \
  < /dev/null > ../events.jsonl 2> ../stderr.log
```

An initial CLI start failed before agent execution because the filesystem was read-only. Retrying with permission for CLI session writes produced this first completed response; the agent sandbox remained read-only.

> This change adds personal pins to the saved-article library. When the library is opened, pinned articles appear first and their labels begin with “Pinned: ”. Within both groups—pinned and unpinned—articles keep their original saved order.
>
> Pins belong to the signed-in member, so one person’s choices don’t affect anyone else’s library. Removing a pin returns the article to its normal place among unpinned articles. Pinning only affects articles already saved in that member’s library; it doesn’t add an article to the library.

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

Recorded with skill v0.5. This unedited response is included as a historical example, not as output from the current release.

> This adds pinning to the saved-article library. Pinned articles now appear at the top and carry a “Pinned:” label. Each member has their own pins.
>
> Previously, articles appeared in the order they were saved. That order is still kept within the pinned and unpinned groups. Unpinning an article leaves it saved and puts it back among the unpinned articles.

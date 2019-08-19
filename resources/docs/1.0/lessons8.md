# Episodes from 55 to 64

- [65-Creating Favorite Component - Part 2 of 3 (Event Handler)](#section-1)
- [66-Creating Favorite Component - Part 3 of 3 (Authenticating the button)](#section-2)
- [67-Creating Accept Answer Component - Part 1 of 2 (from button into Vue component)](#section-3)
- [68-Creating Accept Answer Component - Part 2 of 2 (event handler)](#section-4)
- [69-Rewriting The Authorization Logic - Part 1 of 2 (Core authorization)](#section-5)
- [70-Rewriting The Authorization Logic - Part 2 of 2 (Refactoring)](#section-6)
- [71-Creating Vote Component - Part 1 of 3 (From blade to Vue Component)](#section-7)
- [72-Creating Vote Component - Part 2 of 3 (Event Handling)](#section-8)
- [73-Creating Vote Component - Part 3 of 3 (Fixing issues)](#section-9)
- [74-Creating Vue Answers Component](#section-10)

<a name="section-1"></a>

## Episode-65 Creating Favorite Component - Part 2 of 3 (Event Handler)

`1` - Edit `resources/js/components/Favorite.vue`

```html
<template>
  <a title="Click to mark favorite question (Click agan to undo)"
    :class="classes"
    @ click.prevent="toggle"
  >
...
</template>
```

js part

```js
...
export default {
  props: ["question"],
  data() {
    return {
      isFavorited: this.question.is_favorited,
      count: this.question.favorites_count,
      signedIn: true,
      id: this.question.id
    };
  },
  computed: {
    classes() {
      return [
        "favorite",
        "mt-2",
        !this.signedIn ? "off" : this.isFavorited ? "favorited" : ""
      ];
    },
    endpoint() {
      return `/questions/${this.id}/favorites`;
    }
  },
  methods: {
    toggle() {
      if (!this.signedIn) {
        this.$toast.warning(
          "Please login to favorite this question",
          "Warning",
          {
            timeout: 3000,
            position: "bottomLeft"
          }
        );
        return;
      }
      this.isFavorited ? this.destroy() : this.create();
    },
    destroy() {
      axios.delete(this.endpoint).then(res => {
        this.count--;
        this.isFavorited = false;
        this.$toast.success("Removed from favorites", "Success", {
          timeout: 3000
        });
      });
    },
    create() {
      axios.post(this.endpoint).then(res => {
        this.count++;
        this.isFavorited = true;
        this.$toast.success("Added to favorites", "Success", {
          timeout: 3000
        });
      });
    }
  }
};
...
```

`2` - Edit `app/Http/Controllers/FavoritesController.php`

```php
...
public function store(Question $question)
{
    $question->favorites()->attach(auth()->id());

    if (request()->expectsJson()) {
        return response()->json(null, 204);
    }
    return back();
}

public function destroy(Question $question)
{
    $question->favorites()->detach(auth()->id());
    if (request()->expectsJson()) {
        return response()->json(null, 204);
    }
    return back();
}
...
```

<a name="section-2"></a>

## Episode-66 Creating Favorite Component - Part 3 of 3 (Authenticating the button)

`1` - `resources/views/layouts/app.blade.php`

```php
...
<body>
...
<script>
  window.Auth = {!! json_encode([
    'signedIn' => Auth::check(),
    'user' => Auth::user()
]) !!}
</script>
<script src="{{ asset('js/app.js') }}"></script>
</body>
...
```

`2` - Edit `resources/js/components/Favorite.vue`

```js
...
data() {
  return {
    isFavorited: this.question.is_favorited,
    count: this.question.favorites_count,
    id: this.question.id
  };
},
computed: {
  ...,
  signedIn() {
    return window.Auth.signedIn;
  }
}
...
```
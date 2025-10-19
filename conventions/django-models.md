Django: Model definition best practices
=======================================


# Ten do-nots

1. Do not relay on ORM feature to create multiple models with the same name, controlling its' table and relations naming.
2. Do not rush into `is_deleted` field with `on_delete=PROTECT` for all models, especially overriding manager's methods like `all`, `filter`, etc.
3. Do not allow duality in filtering.
4. Do not ignore the convenience of using the database directly, especially for the sake of technical savings.
5. Do not follow specific ORMs' best practices, follow common best practices.
6. Do not build algorithms tied on db-records.
7. Do not do anything in the database that is not reflected in the code.
8. Do not be shy to denormalize to stay in domain logics and not finish with CQRS.
9. Do not edit migrations without an extreme exceptional need.
10. Do not use DBMS' specific features if you allow/assume vendor changes, on-board delivery, or testing using common universal practices (in-memory SQLite).
    

# How-tos

## 1. Naming models and db-tables

Django allows create multiple models with the same class name with db-table names prefixed with app name, like [`users.Group`](http://users.Group) —&gt; `users_group`, [`posts.Group`](http://posts.Group) —&gt; `posts_group`, etc. It's ok in db, but it brings confusion in the code. It also makes model migration between apps much harder. So,
1. Give model name an app prefix (if needed, or potentially would be). In the example above — better names would be `UserGroup` and `PostGroup`.
2. Specify db-table name explicitly in `Meta` — just translate its' name to the snake-case.
3. Describe M2M fields in 'secondary-table'. Bidirectional solution is also possible: [https://stackoverflow.com/a/9341455/4117781](https://stackoverflow.com/a/9341455/4117781).
4. Give FKs and M2Ms fields logical names, not technical, e.g.: `author`, not `user`. Tech name is for explicit m2m-tables describings.
5. For m2m-fields: use pattern `model_name__field_name`, since 2 tables can have an m2m relation for different purposes.
6. In case separate m2m-table with `through`, specify tables' name in `Meta` with the same pattern `model_name__field_name`.
7. Always specify `default_related_name` in models' `Meta` — just its' name in snake case in plural form.
8. In case default can not be used (multiple FK or M2M) specify fields' `related_name` using pattern `field_name_model_name_plural`.
9. Specify `db_table_comment`, `verbose_name_plural`, `default_related_name` in `Meta`.

Complex example following all steps above:

```python
class Post(models.Model):
	author = models.ForeignKey(User)
	liked = models.ManyToManyField(User, db_table='post__liked',  related_name='liked_posts')
	shared = models.ManyToManyField(User, db_table='post__shared', related_name='shared_posts')
	viewed = models.ManyToManyField(User, throught=PostViewed, related_name='viewed_posts')

    class Meta:
	    db_table = 'post'
	    db_table_comment = 'Post'
	    verbose_name = 'Post'
	    verbose_name_plural = 'Posts'
	    default_related_name = 'posts'


class PostViewed(models.Model):
	user = models.ForeignKey(User)
	post = models.ForeignKey(Post)

	class Meta:
		db_table='post__viewed'


class Comment(models.Model):
	author = models.ForeignKey(User)
	post   = models.ForeignKey(Post)
	text   = models.TextField()

	class Meta:
		db_table='post_comment'
	    db_table_comment = 'Post Comment'
	    verbose_name = 'Post Comment'
	    verbose_name_plural = 'Post Comments'
	    default_related_name = 'comments'
```

> Notice, that m2m-table need only `db_table` to be specified. Also, technically `Comment` — is also an 'm2m-table' between `User` and `Post`, but we see it as a separate logical entity, with an another way of access that relation.

## 2. Handling objects deletion

The worst practice, as it was mentioned earlier:
> Do not rush into `is_deleted` field with `on_delete=PROTECT` for all models, especially overriding manager's methods like `all`, `filter`, etc.

* Sometimes garbage is garbage.
* And there are could also testing purposes, when we have to add and clean some records, even if they considered critical in production.
* `is_deleted` brings no semantic load, it could be `is_archived`, `is_published`.
* `on_delete=CASCADE` will also clear some historical records we could need later.

So, it could depend on case, but common good practive would be:

```python
class Task(models.Model):
	subtask = models.ForeignKey(Task, null=True, on_delete=models.SET_NULL)
	is_active = models.BooleanField(default=False, db_default=False)
```

And querying it simply like:

```python
tasks = Task.objects.filter(is_active=True)
```

## 3. Defaults

1. If it has `default`, it should also have `db_default`.
    

## 4. BooleanField

1. Always set `default`, never `null=True`.
2. Also set `db_default`.

So, in minimal it's like:

```python
is_active = models.BooleanField(default=False, db_default=False)
```

## 5. DateTimeField

1. `default=`[`timezone.now`](http://timezone.now) — if we want date to be prepolated when page/form is rendered. Notice — [`timezone.now`](http://timezone.now) goes without call literals, it's not [`timezone.now`](http://timezone.now)`()`:

```python
from django.utils import timezone

class MyModel(models.Model):
    created_at = models.DateTimeField(default=timezone.now)
```

2. `auto_now_add=True` — if we want date to be prepolated when page/form is submitted, and not managed by any user (even admin). In that case also consider adding `db_default=Now()` (with call literals):

```python
from django.db.models.functions import Now

class MyModel(models.Model):
    created_at = models.DateTimeField(auto_now_add=True, db_default=Now())
```

3. `auto_now=True` — if we want date to be updated each time on form edited (also could not be managed by anyone).

## 6. CharField and TextField

There is also a recommendation in Django docs: "Avoid using null on string-based fields such as CharField and TextField. The Django convention is to use an empty string, not NULL" [https://docs.djangoproject.com/en/5.2/ref/models/fields/#null](https://docs.djangoproject.com/en/5.2/ref/models/fields/#null)
Looks ok for `TextField`, but very rarely actually needed, so let's omit it. But it's totally wrong for `CharField` in practice. `CharField`s are either required (title, slug, username, email), either are storing fixed values via `TextChoices`, either they are just storing notes, that we are not interesting in filtering/analysing. So it's completely safe stay with `null=True`.
1. Avoid `default=''` for `CharField` (and probably `TextField`), stay with `null=True`.
2. It's ok to set `max_length=255` if contents in unclear.

So, that's completely legal:

```python
text = models.CharField(max_length=255, null=True, blank=True)
```

## 7. Choices

> Do not build algorithms tied on db-records. Do not ignore the convenience of using the database directly, especially for the sake of technical savings.

1. Use `TextChoices` over `IntegerChoices` in most common cases.
2. Name choices class in plural.
3. Make values uppercase to make it clear that this is not just a regular string with any possible value.
4. Describe choices class inside model unless it is used in other models' choices.
    
So, this:

```python
class MyModel(models.Model):
	class Statuses:
		CREATED = 'CREATED', 'Created'
		COMPLETED = 'COMPLETED', 'Completed'

	status = models.CharField(max_length=20, choices=Statuses, default=Statuses.CREATED, db_default=Statuses.CREATED)
```

Is better than this:

```python
class Status:
	CREATED = 1, 'Created'
	COMPLETED = 2, 'Completed'


class MyModel(models.Model):
	status = models.SmallPositiveIntegerField(choices=Status)
```

Worst is:

```python
class Status(models.Model):
	name = models.ChatField(max_length=255)


class MyModel(models.Model):
	status = models.ForeignKey(Status)
```

> Same for rights/permissions.

---

Article will be supplemented over time
## TREE-DATABASE-STRUCTURE
<br>
<p align='center'>
<img src="https://github.com/Jauharmuhammed/README-Template/blob/main/assets/Mockup-website.png" width='70%' >
</p>
<br>

Guide to create a hierarchical categories in a tree structure easily with Django.

- Lorem ipsum dolor sit amet, consectetur adipiscing elit.
- In sit amet cursus ex.
- Maecenas id lacus ut diam placerat lobortis ac quis nisl.
-  Nam tristique vulputate ante at malesuada.
- imperdiet elit posuere ut.

<br>

## 1. Install mptt
#1.1 Package

```
pip install django-mptt
```


#1.2 Add to settings

```
INSTALLED_APPS = (
    'django.contrib.auth',
    # ...
    'mptt',
)
```
## 2. Configure models


```
from django.template.defaultfilters import slugify
from mptt.models import MPTTModel, TreeForeignKey

class Category(MPTTModel):
    category        =   models.CharField(max_length=255,db_index=True)
    parent          =   TreeForeignKey('self',blank=True,null=True,related_name='child',on_delete=models.CASCADE)
    slug            =   models.SlugField(max_length=255,unique=True)
    description     =   models.TextField(blank=True,null=True)
    is_active       =   models.BooleanField(default=True)

    class MPTTMeta:
        order_insertion_by = ['category']
        
    def __str__(self):                           
        full_path = [self.category]            
        k = self.parent
        while k is not None:
            full_path.append(k.category)
            k = k.parent
        return ' -> '.join(full_path[::-1])

```
## 3. create table
```
python manage.py makemigrations <your_app>
python manage.py migrate
```


##4. admin

```
class CategoryAdmin(DraggableMPTTAdmin):
    mptt_indent_field = "category"
    list_display = ('tree_actions', 'indented_title',
                    'related_products_count', 'related_products_cumulative_count')
    list_display_links = ('indented_title',)
    prepopulated_fields = {'slug': ('category',)}
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)

        # Add cumulative product count
        qs = Category.objects.add_related_count(
                qs,
                Product,
                'category',
                'products_cumulative_count',
                cumulative=True)

        # Add non cumulative product count
        qs = Category.objects.add_related_count(qs,
                 Product,
                 'category',
                 'products_count',
                 cumulative=False)
        return qs

    def related_products_count(self, instance):
        return instance.products_count
    related_products_count.short_description = 'Related products (for this specific category)'

    def related_products_cumulative_count(self, instance):
        return instance.products_cumulative_count
    related_products_cumulative_count.short_description = 'Related products (in tree)'
    
admin.site.register(Category,CategoryAdmin)
```

## 5. Template


```
{% load mptt_tags %}
<ul>
    {% recursetree genres %}
        <li>
            {{ node.name }}
            {% if not node.is_leaf_node %}
                <ul class="children">
                    {{ children }}
                </ul>
            {% endif %}
        </li>
    {% endrecursetree %}
</ul>
```



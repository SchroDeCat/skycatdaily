# Resolve Length Home Page
## Motivation
After deploying "Hexo + next" on my server, I find it really bothering that the home page doesn't conduct auto-excerption on the articles like ghost. However, i can be artificially set in the configuration of next theme. 
## Configuration
```bash
# Automatically Excerpt. Not recommand.
# Please use <!-- more --> in the post to control excerpt accurately.
auto_excerpt:
  enable: false
  length: 150
```

## Ref
[Origin Tutorial](https://www.jianshu.com/p/393d067dba8d)

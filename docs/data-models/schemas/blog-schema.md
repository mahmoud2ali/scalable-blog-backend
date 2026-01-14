## Blog Schema

[User] 1---* [Post] 1---* [Comment]

- User: id, username, email, role, created_at
- Post: id, author_id(User), title, content, created_at
- Comment: id, post_id(Post), user_id(User), content, created_at

## Notes
- One User --> many posts
- One Post --> many comments


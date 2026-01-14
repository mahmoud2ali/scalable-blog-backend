## Chat Schema

`[ChatRoom] 1---* [Message]`
`[User] *---* [ChatRoom]`

- User: id, username
- ChatRoom: id, name, participants(list of Users)
- Message: id, chatRoomId, senderId, content, timestamp

## Notes
- Messages are append-only


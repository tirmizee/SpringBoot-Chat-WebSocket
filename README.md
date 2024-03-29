# SpringBoot-Chat-WebSocket

### Technology stack

- Spring boot
- Apache Kafka
- Mysql
- React
- WebSocket 

### Table

- users
- chat_room
- chat_room_user
- chat_message

### SQL

```sql

SELECT
	R.ID AS ROOM_ID, 
	R.NAME AS ROOM_NAME, 
	R.TYPE,
	U.ID AS USER_ID,
	U.USERNAME AS USERNAME,
	U.IMAGE_URL,
	:USERNAME AS ACTIVE_USER, 
	RU.UN_READ AS UN_READ, 
	(
		SELECT 
			UN_READ 
		FROM CHAT_ROOM_USER
		WHERE ROOM_ID = R.ID 
		AND USER_ID = AU.ID
	) AS ACTIVE_USER_UN_READ, 
	(
		SELECT 
			MESSAGE 
		FROM CHAT_MESSAGE 
		WHERE ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
	) AS LAST_MESSAGE,
	(
		SELECT 
			U.USERNAME 
		FROM CHAT_MESSAGE C
		INNER JOIN USERS U ON C.USERS_ID = U.ID 
		WHERE C.ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
	) AS LAST_SENDER, 
	(
		SELECT CREATE_DATE 
		FROM CHAT_MESSAGE 
		WHERE ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
	) AS LAST_SEND
FROM CHAT_ROOM R
INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
INNER JOIN USERS U ON U.ID = RU.USER_ID
INNER JOIN USERS AU ON AU.USERNAME = :USERNAME
WHERE RU.ROOM_ID IN (
	SELECT 
		R.ID 	
	FROM CHAT_ROOM R
	INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
	INNER JOIN USERS U ON U.ID = RU.USER_ID
	WHERE U.USERNAME = :USERNAME
	AND R.TYPE = 'P'
) AND U.USERNAME <> :USERNAME

```

...

```sql

SELECT T.* FROM (
	SELECT
		NULL AS ROOM_ID, 
		NULL AS ROOM_NAME, 
		'P' AS TYPE,
		U.ID AS USER_ID,
		U.USERNAME AS USERNAME,
		NULL AS IMAGE_URL,
		'pratya' AS ACTIVE_USER,
		0 AS UN_READ,
        0 AS ACTIVE_USER_UN_READ,
		NULL AS LAST_MESSAGE,
		NULL AS LAST_SENDER, 
		NULL AS LAST_SEND
	FROM USERS U
	WHERE U.ID NOT IN (
		SELECT
			U.ID 
		FROM CHAT_ROOM R
		INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
		INNER JOIN USERS U ON U.ID = RU.USER_ID
		INNER JOIN (
			SELECT 
				U.ID,
				RU.ROOM_ID
			FROM CHAT_ROOM R
			INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
			INNER JOIN USERS U ON U.ID = RU.USER_ID
			WHERE U.USERNAME = 'pratya'
			AND R.TYPE = 'P'
		) T ON T.ROOM_ID = R.ID
	)

	UNION

	SELECT
		R.ID AS ROOM_ID, 
		R.NAME AS ROOM_NAME, 
		R.TYPE,
		U.ID AS USER_ID,
		U.USERNAME AS USERNAME,
		U.IMAGE_URL,
		'pratya' AS ACTIVE_USER, 
		RU.UN_READ AS UN_READ, 
		(
			SELECT 
				UN_READ 
			FROM CHAT_ROOM_USER
			WHERE ROOM_ID = R.ID 
			AND USER_ID = AU.ID
		) AS ACTIVE_USER_UN_READ, 
		(
			SELECT 
				MESSAGE 
			FROM CHAT_MESSAGE 
			WHERE ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
		) AS LAST_MESSAGE,
		(
			SELECT 
				U.USERNAME 
			FROM CHAT_MESSAGE C
			INNER JOIN USERS U ON C.USERS_ID = U.ID 
			WHERE C.ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
		) AS LAST_SENDER, 
		(
			SELECT CREATE_DATE 
			FROM CHAT_MESSAGE 
			WHERE ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
		) AS LAST_SEND
	FROM CHAT_ROOM R
	INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
	INNER JOIN USERS U ON U.ID = RU.USER_ID
    INNER JOIN USERS AU ON AU.USERNAME = 'pratya'
	WHERE RU.ROOM_ID IN (
		SELECT 
			R.ID 	
		FROM CHAT_ROOM R
		INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
		INNER JOIN USERS U ON U.ID = RU.USER_ID
		WHERE U.USERNAME = 'pratya'
		AND R.TYPE = 'P'
	) AND  U.USERNAME <> 'pratya'
) T       		

```

...

```sql

SELECT
	R.ID AS ROOM_ID, 
	R.NAME AS ROOM_NAME, 
	R.TYPE,
	U.ID AS USER_ID,
	U.USERNAME AS USERNAME,
	U.IMAGE_URL,
	:USERNAME AS ACTIVE_USER,
	T.UN_READ AS UN_READ, 
    RU.UN_READ AS ACTIVE_USER_UN_READ, 
	(
		SELECT 
			MESSAGE 
		FROM CHAT_MESSAGE 
		WHERE ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
	) AS LAST_MESSAGE,
	(
		SELECT 
			U.USERNAME 
		FROM CHAT_MESSAGE C
		INNER JOIN USERS U ON C.USERS_ID = U.ID 
		WHERE C.ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
	) AS LAST_SENDER, 
	(
		SELECT CREATE_DATE 
		FROM CHAT_MESSAGE 
		WHERE ID = (SELECT MAX(ID) FROM CHAT_MESSAGE WHERE CHAT_ROOM_ID = R.ID ) 
	) AS LAST_SEND
FROM CHAT_ROOM R
INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
INNER JOIN USERS U ON U.ID = RU.USER_ID
INNER JOIN (
	SELECT 
		RU.ROOM_ID,
	RU.UN_READ
	FROM CHAT_ROOM R
	INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
	INNER JOIN USERS U ON U.ID = RU.USER_ID
	WHERE U.USERNAME = :TO_USERNAME
	AND R.TYPE = 'P'
) T ON T.ROOM_ID = R.ID
WHERE U.USERNAME = :USERNAME
AND R.TYPE = 'P'

```

### Reference

- https://dev.to/subhransu/realtime-chat-app-using-kafka-springboot-reactjs-and-websockets-lc
- https://www.toptal.com/java/stomp-spring-boot-websocket
- https://helptechcommunity.wordpress.com/2020/01/28/websocket-chat-application-using-spring-boot-and-react-js/
- https://www.devglan.com/spring-boot/spring-boot-websocket-example?baseUrl=https%3A%2F%2Fwww.devglan.com%2F
- https://stackoverflow.com/questions/50573461/spring-websockets-authentication-with-spring-security-and-keycloak
- https://stackoverflow.com/questions/45405332/websocket-authentication-and-authorization-in-spring
- https://github.com/Rob-Leggett/angular_websockets_security/blob/master/security/src/main/java/au/com/example/security/spring/security/interceptor/TokenSecurityChannelInterceptor.java
- https://github.com/jmesnil/stomp-websocket (Stomp)
- https://github.com/stomp-js/stompjs (Stomp)
- https://stomp-js.github.io/stomp-websocket/codo/extra/docs-src/Usage.md.html
- https://github.com/kingofthestack/react-chat-window

# SpringBoot-WebSocket

### SQL

```sql

SELECT T.* FROM (
	SELECT
		NULL AS ROOM_ID, 
		NULL AS ROOM_NAME, 
		'P' AS TYPE,
		U.ID AS USER_ID,
		U.USERNAME AS USERNAME,
		NULL AS IMAGE_URL,
		'PRATYA' AS ACTIVE_USER,
		0 AS UN_READ,
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
			WHERE U.USERNAME = 'PRATYA'
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
		'PRATYA' AS ACTIVE_USER,
		RU.UN_READ, 
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
	WHERE RU.ROOM_ID IN (
		SELECT 
			R.ID 	
		FROM CHAT_ROOM R
		INNER JOIN CHAT_ROOM_USER RU ON RU.ROOM_ID = R.ID
		INNER JOIN USERS U ON U.ID = RU.USER_ID
		WHERE U.USERNAME = 'PRATYA'
		AND R.TYPE = 'P'
	) AND  U.USERNAME <> 'PRATYA'
) T

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

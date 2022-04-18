## [HTTP]메세지

### 메세지의 흐름

### 메세지의 각 부분
<ol>
    <li>시작줄
        <ol>
            <li>요청줄</li>
            <li>응답줄</li>
            <li>메서드</li>
            <li>상태코드</li>
            <li>사유 구절</li>
            <li>버전 번호</li>
        </ol>
    </li>
    <li>헤더 블록
        <ol>
            <li>헤더 분류</li>
            <li>일반 헤더</li>
            <li>요청 헤더</li>
            <li>응답 헤더</li>
            <li>Entity 헤더</li>
            <li>확장 헤더</li>
            <li><i>헤더를 여러 줄로 나누기</i></li>
        </ol>
    </li>
    <li>본문(엔터티 본문)</li>
</ol>

### 메세지 문법
<ol>
    <li>요청메세지 형식</li>
    <li>응답메세지 형식</li>
    <ol>
        <li>메서드
            <ol>
                <li>안전한 메서드(Safe Method)</li>
                <li>GET</li>
                <li>HEAD</li>
                <li>PUT</li>
                <li>POST</li>
                <li>TRACE</li>
                <li>OPTIONS</li>
                <li>DELETE</li>
                <li>확장 메서드</li>
            </ol>
        </li>
        <li>요청 URL</li>
        <li>버전</li>
        <li>상태 코드
            <ol>
                <li>100-199: 정보성 상태 코드
                    <ol>
                        <li>클라이언트와 100 Continue</li>
                        <li>서버와 100 Continue</li>
                        <li>프락시와 100 Continue</li>
                    </ol>
                </li>
                <li>200-200: 성공 상태 코드</li>
                <li>300-399: 리다이랙션 상태 코드</li>
                <li>400-499: 클라이언트 에러 상태 코드</li>
                <li>500-599: 서버 에러 상태 코드</li>
            </ol>
        </li>
        <li>사유 구절</li>
        <li>헤더들
            <ol>
                <li>일반 헤더(General Headers)
                    <ol>
                        <li>일반 캐시 헤더</li>
                    </ol>
                </li>
                <li>요청 헤더(Request Headers)
                    <ol>
                        <li>Accept 관련 헤더</li>
                        <li>조건부 요청 헤더</li>
                        <li>요청 보안 헤더</li>
                        <li>프락시 요청 헤더</li>
                    </ol>
                </li>
                <li>응답 헤더(Response Headers)
                    <ol>
                        <li>협상 헤더</li>
                        <li>응답 보안 헤더</li>
                    </ol>
                </li>
                <li>엔터티 헤더(Entity Headers)
                    <ol>
                        <li>콘텐츠 헤더</li>
                        <li>엔터티 캐싱 헤더</li>
                    </ol>
                </li>
                <li>확장 헤더(Extension Headers)</li>
            </ol>
        </li>
        <li>엔터티 본문</li>
    </ol>
</ol>

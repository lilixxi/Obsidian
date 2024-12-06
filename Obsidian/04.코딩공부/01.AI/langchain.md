# 0. 환경변수 설정 

```python
# API KEY를 환경변수로 관리하기 위한 설정 파일
from dotenv import load_dotenv
# API KEY 정보로드
load_dotenv()
```

# 1. 랭스미스 추적 
```python
from langchain_teddynote import logging
# 프로젝트 이름을 입력합니다.
logging.langsmith("CH01-Basic")
logging.langsmith("CH01-Basic", set_enable=False)
```
랭스미스 추적을 하는 이유? 
![[Pasted image 20241206192250.png]]
토큰값, 사용모델 . input,output 등 여러가지 정보를 추적할 수 있다. 

# 2. LLM
```python
from langchain_openai import ChatOpenAI
# 1.기본
llm = ChatOpenAI(
temperature=0.1, # 창의성 (0.0 ~ 2.0)
model_name="gpt-4o-mini", # 모델명
)

# 2. 토큰확률 로그값 
llm_with_logprob = ChatOpenAI(
temperature=0.1, # 창의성 (0.0 ~ 2.0)
max_tokens=2048, # 최대 토큰수
model_name="gpt-4o-mini", # 모델명
).bind(logprobs=True)
```
- invoke : 메타데이터 정보를 포함한 답변 받기 
- stream : stream 방식의 답변 받기 
- MultiModal - invoke/stream : 이미지 인식

# 3. 프롬프트 템플릿(!!)
```python
# PromptTemplate.from_template
#################################
# 1. 간단한 템플릿 
template = "{country}의 수도는 어디인가요?"
#################################
# 2. 복잡한 템플릿
template = """
당신은 미국에사는 영어 선생님입니다. 문장에서 문법적으로 틀리거나 회화적으로 맞지 않는 표현을 자연스러운 회화표현으로 고치세요.
또한 왜 그런 회화표현을 사용했는지 한국말로 설명해주고, 영어로 물어본 문장에 대해 대화하듯이 대답하세요.
문장이 올바르다면 대답만 하세요.
양식은 [FORMAT]을 참고하여 작성해 주세요.
#FORMAT:
- 만든 문장 :{question}
- 고친 표현 :
- 설명 :
- 답변 :
"""

prompt_template = PromptTemplate.from_template()

prompt = prompt_template.format(country="대한민국")
#################################
# 3. input_variables 을 이용해 변수 지정도 가능 
prompt = PromptTemplate(
template=template,
input_variables=["country"],
)
prompt.format(country="대한민국")
#################################
# 4. partial_variables 로 중간 전달값 넣기 
prompt = PromptTemplate(
template=template,
input_variables=["country1"],
#################################
# 5. partial_variables : 입력받을 변수를 중간과정에서 확정시
partial_variables={
"country2": "미국" # dictionary 형태로 partial_variables를 전달 },)
# 새로운 값으로 덮을 수 있다 
chain.invoke({"country1": "대한민국", "country2": "호주"}).content
#################################
# 6. 프롬프트를 파일로 지정 
prompt = load_prompt("prompts/fruit_color.yaml", encoding="utf-8")
# 체인 생성 코드 
# 호출 
#################################
# 7. ChatPromptTemplate
chat_prompt = ChatPromptTemplate.from_template("{country}의 수도는 어디인가요?")
# system / ai / human 으로 나누어 역할 지정 가능 
chat_template = ChatPromptTemplate.from_messages(
[
# role, message
# 처음 role 넣기 (system,human,ai 중 하나)
("system", "당신은 친절한 AI 어시스턴트입니다. 당신의 이름은 {name} 입니다."),
# system 전역설정(대화전체) : AI 역할 지정
("human", "반가워요!"),
("ai", "안녕하세요! 무엇을 도와드릴까요?"),
("human", "{user_input}"),])
# 체인 생성 코드 
# 호출 
#################################
# 8. MessagesPlaceholder
chat_prompt = ChatPromptTemplate.from_messages(
[("system",
"당신은 요약 전문 AI 어시스턴트입니다. 당신의 임무는 주요 키워드로 대화를 요약하는 것입니다.",),
MessagesPlaceholder(variable_name="conversation"), # 나중에 채워질것을 예상 / 대화 기록
("human", "지금까지의 대화를 {word_count} 단어로 요약합니다."),])

formatted_chat_prompt = chat_prompt.format(
word_count=5,
conversation=[
("human", "안녕하세요! 저는 오늘 새로 입사한 테디 입니다. 만나서 반갑습니다."),
("ai", "반가워요! 앞으로 잘 부탁 드립니다."),],)
# 체인 생성 코드 
# 호출 
#################################
# 9. FewShotPromptTemplate
examples = [
{
"question": "스티브 잡스와 아인슈타인 중 누가 더 오래 살았나요?",
"answer": """이 질문에 추가 질문이 필요한가요: 예.
추가 질문: 스티브 잡스는 몇 살에 사망했나요?
중간 답변: 스티브 잡스는 56세에 사망했습니다.
추가 질문: 아인슈타인은 몇 살에 사망했나요?
중간 답변: 아인슈타인은 76세에 사망했습니다.
최종 답변은: 아인슈타인
""",},
{
"question": "네이버의 창립자는 언제 태어났나요?",
"answer": """이 질문에 추가 질문이 필요한가요: 예.
추가 질문: 네이버의 창립자는 누구인가요?
중간 답변: 네이버는 이해진에 의해 창립되었습니다.
추가 질문: 이해진은 언제 태어났나요?
중간 답변: 이해진은 1967년 6월 22일에 태어났습니다.
최종 답변은: 1967년 6월 22일
""",},
{
"question": "율곡 이이의 어머니가 태어난 해의 통치하던 왕은 누구인가요?",
"answer": """이 질문에 추가 질문이 필요한가요: 예.
추가 질문: 율곡 이이의 어머니는 누구인가요?
중간 답변: 율곡 이이의 어머니는 신사임당입니다.
추가 질문: 신사임당은 언제 태어났나요?
중간 답변: 신사임당은 1504년에 태어났습니다.
추가 질문: 1504년에 조선을 통치한 왕은 누구인가요?
중간 답변: 1504년에 조선을 통치한 왕은 연산군입니다.
최종 답변은: 연산군
""",},]

# 형식 지정 
example_prompt = PromptTemplate.from_template(
"Question:\n{question}\nAnswer:\n{answer}"
)

prompt = FewShotPromptTemplate(
# 예제 넣어주기 
examples=examples,
# 프롬프트 형식 지정 
example_prompt=example_prompt,
# suffix : 새로들어온 사용자의 질문
suffix="Question:\n{question}\nAnswer:", 
# 받을 변수 
input_variables=["question"],
)
final_prompt = prompt.format(question=question)
# 호출 
#################################
# 10. 유사도 기반 으로 가장 가까운 예제만 러닝 
example_selector = SemanticSimilarityExampleSelector.from_examples(
# 여기에는 선택 가능한 예시 목록이 있습니다.
# 예시 
examples,
# 임베딩
OpenAIEmbeddings(),
# 벡터
Chroma
# 가져올 문장 수 
k=1,)

# 가장 유사한 문장 가져오기 
selected_examples = example_selector.select_examples({"question": question})
prompt = FewShotPromptTemplate(
example_selector=example_selector,
example_prompt=example_prompt,
suffix="Question:\n{question}\nAnswer:",
input_variables=["question"],
)
# 체인 생성 코드 
#################################
# 11. FewShotChatMessagePromptTemplate
examples = [

{

"instruction": "역할부여",
"input": "상황부여",
"answer": """
회의록: 내용
일시: 내용
장소: 내용
참석자: 김수진 (마케팅 팀장), 박지민 (디지털 마케팅 담당자), 이준호 (소셜 미디어 관리자)
1. 개회
- 내용
2. 시장 동향 개요 (김수진)
- 내용
3. 디지털 마케팅 전략 (박지민)
- 내용
4. 소셜 미디어 캠페인 (이준호)
- 내용
5. 종합 논의
- 내용
6. 마무리
- 내용
""",

},
{비슷한 다른 예시 ,},
{비슷한 다른 예시 ,},]

few_shot_prompt = FewShotChatMessagePromptTemplate(
# 함수 (위와 동일)
example_selector=example_selector,
# 예시 프롬프트 (위와 동일) -> 
example_prompt=example_prompt,

# custom_selector 
custom_fewshot_prompt = FewShotChatMessagePromptTemplate(
example_selector=custom_selector, # 커스텀 예제 선택기 사용 !(디폴트 instruction)
example_prompt=example_prompt, # 예제 프롬프트 사용
)

custom_prompt = ChatPromptTemplate.from_messages(
[("system","You are a helpful assistant.",),
custom_fewshot_prompt,
("human", "{instruction}\n{input}"),])

# question 을 바꿔서 실행하면 custom_selector 가 instruction 내용과 비슷한 문서를 가져와서 문장을 만들어준다 
question = {
"instruction": "문장을 교정해 주세요",
"input": "회사는 올해 매출이 증가할 것으로 예상한다. 새로운 전략이 잘 작동하고 있다.",
}
# 체인 생성 코드 
# 호출 
```
변수를 지정하고 그 변수를 반드시 넣어줘야 한다 
* input_variables : 변수 지정
* partial_variables : 중간 전달값 넣기 
* ChatPromptTemplate : 대화형식 
* MessagesPlaceholder : 어떤 역할을 사용해야 할지 확실하지 않거나 서식 지정 중에 메시지 목록을 삽입하려는 경우
* FewShotPromptTemplate : 약간의 예제를 주어 ai 가 따라할수 있게 하는 것 
# 4. 체인 생성
```python
# 기본 구성
chain = prompt | model | output_parser

# RunnablePassthrough 이용 
runnable_chain = {"num": RunnablePassthrough()} | prompt | ChatOpenAI()

# RunnableLambda 이용
chain = (
{"today": RunnableLambda(get_today), "n": RunnablePassthrough()}
| prompt
| llm
| StrOutputParser()
)

# itemgetter 이용
chain = (
{"today": RunnableLambda(get_today), "n": itemgetter("n")}
# get_today 함수 사용 / itemgetter 로 n 을 꺼내고 전달 
| prompt
| llm
| StrOutputParser()
))
```
Runnable : 데이터 가공 및 전달 
* RunnablePassthrough : 가공 없이 Pass 그대로 출력, 원래의 데이터를 보고 싶을때 
* RunnableLambda : 함수를 넣어 가공 
* itemgetter : 특정 키 추출 
## 배치단위 실행 
```python
# 여러가지를 병렬처리로 실행 
# 1. 간단한 병렬처리 
chain.batch([{"topic": "ChatGPT"}, {"topic": "Instagram"}])

# 2. 여러가지 병렬처리
chain.batch(
[
{"topic": "ChatGPT"},
{"topic": "Instagram"},
{"topic": "멀티모달"},
{"topic": "프로그래밍"},
{"topic": "머신러닝"},
],
config={"max_concurrency": 3},
)

# 체인 병렬연결 
chain1 = (
PromptTemplate.from_template("{country} 의 수도는 어디야?")
| model
| StrOutputParser()
)
# {country} 의 면적을 물어보는 체인을 생성합니다.
chain2 = (
PromptTemplate.from_template("{country} 의 면적은 얼마야?")
| model
| StrOutputParser()
)

# 위의 2개 체인을 동시에 생성하는 병렬 실행 체인을 생성합니다.
combined = RunnableParallel(capital=chain1, area=chain2)
# 배치 처리를 수행합니다.
chain1.batch([{"country": "대한민국"}, {"country": "미국"}])
```
## 비동기 스트림 
```python
# async
# 비동기 스트림을 사용하여 'YouTube' 토픽의 메시지를 처리합니다.
async for token in chain.astream({"topic": "YouTube"}):
# 메시지 내용을 출력합니다. 줄바꿈 없이 바로 출력하고 버퍼를 비웁니다.
print(token, end="", flush=True)
my_process = chain.ainvoke({"topic": "NVDA"})
await my_process
```

## 비동기 + 배치 
```python
# 배치
my_abatch_process = chain.abatch(
[{"topic": "YouTube"}, {"topic": "Instagram"}, {"topic": "Facebook"}])
# 비동기 
await my_abatch_process
```
```python

```
```python

```

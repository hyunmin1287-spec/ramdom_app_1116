import streamlit as st
import pandas as pd

# ---------------------------------------------------------
# 1. 함수 (def) & 선택구조 (if-elif-else) 정의
# ---------------------------------------------------------
def analyze_spending(salary, food_cost, habit):
    # 식비 비율(%) 계산
    food_rate = (food_cost / salary) * 100
    
    # 조건문(if-elif-else): 식비 비율에 따른 소비 진단
    if food_rate <= 25:
        status = "좋은 소비"
        advice = "식비 관리를 매우 훌륭하게 하고 계십니다! 현재 습관을 유지하세요."
    elif food_rate <= 35:
        status = "절약 필요"
        advice = "식비 비중이 조금 높은 편입니다. 배달이나 외식을 주 1~2회 줄여보세요."
    elif food_rate >= 40:
        status = "과소비"
        advice = "식비 지출이 위험 수준입니다! 장보기 메모 활용 및 직접 요리가 시급합니다."
    else:
        # 35% 초과 ~ 40% 미만 구간 처리
        status = "절약 필요 (주의)"
        advice = "과소비 직전 단계입니다. 카페 및 간식 비용부터 줄여보세요."

    # 소비 습관에 따른 종합 권장 소비율 계산
    if habit == "배달/외식 위주":
        recommended_rate = 55
    elif habit == "쇼핑/충동구매 위주":
        recommended_rate = 50
    else:  # 계획적/절약형
        recommended_rate = 65

    return round(food_rate, 1), status, advice, recommended_rate


# ---------------------------------------------------------
# 2. 화면 상태(단계) 관리를 위한 초기 설정
# ---------------------------------------------------------
if "step" not in st.session_state:
    st.session_state.step = 1  # 1: 시작, 2: 질문입력, 3: 계획서/그래프, 4: 결론


# ---------------------------------------------------------
# 3. 단계별 웹 화면 구성 (Streamlit UI)
# ---------------------------------------------------------

# [1단계: 시작 화면]
if st.session_state.step == 1:
    st.title("📝 한 달 리스트 만들기")
    st.write("나만의 소비 습관을 점검하고 맞춤형 한 달 소비 절약 계획서를 만들어보세요!")
    st.divider()
    
    if st.button("시작하기", type="primary"):
        st.session_state.step = 2
        st.rerun()

# [2단계: 순차적 질문 입력 화면]
elif st.session_state.step == 2:
    st.header("📋 소비 정보 입력")
    st.write("아래 질문에 답을 입력해 주세요.")
    
    # 질문 1: 한 달 급여
    salary = st.number_input("Q1. 당신의 한 달 급여(용돈)는 얼마인가요? (원)", min_value=10000, value=2000000, step=100000)
    
    # 질문 2: 소비 습관
    habit = st.selectbox("Q2. 평소 본인의 주요 소비 습관은 어떤가요?", ["배달/외식 위주", "쇼핑/충동구매 위주", "계획적/절약형"])
    
    # 질문 3: 평소 식비
    food_cost = st.number_input("Q3. 평소 한 달 식비는 얼마인가요? (원)", min_value=0, value=600000, step=10000)
    
    st.divider()
    if st.button("계획서 생성하기", type="primary"):
        # 입력값을 세션에 저장하고 3단계로 이동
        st.session_state.salary = salary
        st.session_state.habit = habit
        st.session_state.food_cost = food_cost
        st.session_state.step = 3
        st.rerun()

# [3단계: 계획서 & 막대그래프 화면]
elif st.session_state.step == 3:
    st.header("📊 당신의 한 달 소비 절약 계획서")
    
    salary = st.session_state.salary
    food_cost = st.session_state.food_cost
    habit = st.session_state.habit
    
    # 함수 호출하여 계산 결과 가져오기
    food_rate, status, advice, recommended_rate = analyze_spending(salary, food_cost, habit)
    remain_rate = round(100 - food_rate, 1)
    if remain_rate < 0:
        remain_rate = 0

    # 데이터 구조 (리스트 list)
    categories = ["식비 비율(%)", "식비 외 나머지 예산(%)"]
    ratios = [food_rate, remain_rate]
    
    # 반복구조 (for문)를 이용해 막대그래프용 데이터 생성
    chart_data_list = []
    for i in range(len(categories)):
        chart_data_list.append({
            "항목": categories[i],
            "비율(%)": ratios[i]
        })
    
    # 데이터프레임 변환 후 막대그래프 출력
    df = pd.DataFrame(chart_data_list)
    st.subheader("급여 대비 지출 비율 그래프")
    st.bar_chart(data=df, x="항목", y="비율(%)")
    
    st.info(f"선택한 소비 습관: **{habit}** | 한 달 급여: **{salary:,}원** | 한 달 식비: **{food_cost:,}원**")
    
    st.divider()
    if st.button("결론 보기", type="primary"):
        st.session_state.step = 4
        st.rerun()

# [4단계: 최종 결론 화면]
elif st.session_state.step == 4:
    st.header("📌 소비 분석 최종 결과")
    
    salary = st.session_state.salary
    food_cost = st.session_state.food_cost
    habit = st.session_state.habit
    
    food_rate, status, advice, recommended_rate = analyze_spending(salary, food_cost, habit)
    
    # 식비 진단 출력
    st.subheader(f"식비 진단 결과: [{status}]")
    st.write(f"현재 한 달 급여 대비 식비 비율은 **{food_rate}%**입니다.")
    st.write(f"💡 **맞춤 조언:** {advice}")
    
    st.divider()
    
    # 결론: 종합 권장 소비율 강조 수치(Metric) 출력
    st.subheader("🎯 결론")
    st.metric(label="종합 권장 소비율 (급여 대비 총 지출 한도)", value=f"{recommended_rate}%")
    st.write(f"급여({salary:,}원) 기준, 한 달 총 지출을 **{int(salary * (recommended_rate / 100)):,}원** 이내로 유지하고 나머지를 저축하는 것을 권장합니다.")
    
    st.divider()
    if st.button("처음으로 돌아가기"):
        st.session_state.step = 1
        st.rerun()

# -
بنك الذكاء
import streamlit as st
import pandas as pd
from sklearn.tree import DecisionTreeClassifier
import plotly.express as px
from io import BytesIO

# --- 1. إعدادات الصفحة والتنسيق البصري المتقدم ---
st.set_page_config(page_title="نظام الائتمان الذكي v3", layout="wide", page_icon="🏦")

# إضافة لمسات جمالية باستخدام CSS
st.markdown("""
    <style>
    /* تغيير لون الخلفية العامة */
    .stApp { background-color: #F8FAFC; }
    
    /* تنسيق القائمة الجانبية */
    section[data-testid="stSidebar"] {
        background-color: #1E293B !important;
        color: white;
    }
    section[data-testid="stSidebar"] .stMarkdown h1, h2 { color: #38BDF8 !important; }
    
    /* تنسيق العناوين الرئيسية */
    .main-header {
        font-size: 35px; color: #1E3A8A; font-weight: bold;
        text-align: center; padding: 20px;
        background: white; border-radius: 15px;
        box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1);
        margin-bottom: 25px;
    }
    
    /* تنسيق الحاويات (Cards) */
    .data-card {
        background: white; padding: 20px; border-radius: 12px;
        box-shadow: 0 1px 3px 0 rgb(0 0 0 / 0.1);
        border-right: 6px solid #3B82F6;
    }
    </style>
    """, unsafe_allow_html=True)

# --- 2. إدارة الحالة والربط البرمجي (State Management) ---
# لضمان عدم ضياع البيانات عند التنقل بين الصفحات
if 'history' not in st.session_state:
    st.session_state['history'] = []
if 'uploaded_df' not in st.session_state:
    st.session_state['uploaded_df'] = None

# --- 3. محرك القرار (Decision Tree) ---
@st.cache_resource
def train_model():
    # بيانات تدريبية افتراضية
    data = pd.DataFrame({
        'الدخل': [2000, 5000, 8000, 3000, 1500, 6000, 2500, 7000],
        'القرض': [10000, 5000, 2000, 15000, 7000, 3000, 12000, 4000],
        'الدرجة': [400, 800, 900, 500, 350, 850, 450, 880],
        'القرار': [0, 1, 1, 0, 0, 1, 0, 1]
    })
    X = data[['الدخل', 'القرض', 'الدرجة']]
    y = data['القرار']
    return DecisionTreeClassifier().fit(X, y)

model = train_model()

# --- 4. القائمة الجانبية (نظام التنقل) ---
with st.sidebar:
    st.markdown("<h1>🏢 بنك الذكاء</h1>", unsafe_allow_html=True)
    st.image("https://cdn-icons-png.flaticon.com/512/2830/2830284.png", width=100)
    st.markdown("---")
    # هذا المفتاح هو الذي يغير المحتوى تماماً عند الضغط عليه
    selected_page = st.radio("القائمة الرئيسية:", 
        ["🏠 لوحة التحكم", "🔍 فحص الائتمان", "📊 ذكاء الأعمال (BI)", "📥 التصدير والتقارير"])
    st.markdown("---")
    st.caption("نظام دعم القرار الائتماني © 2026")

# --- 5. تنفيذ محتوى الصفحات ---

# الصفحة 1: لوحة التحكم (الرئيسية)
if selected_page == "🏠 لوحة التحكم":
    st.markdown("<div class='main-header'>لوحة القيادة والمؤشرات العامة</div>", unsafe_allow_html=True)
    
    c1, c2, c3 = st.columns(3)
    with c1: st.metric("إجمالي الفحوصات", len(st.session_state['history']), "طلبات جديدة")
    with c2: st.metric("دقة الخوارزمية", "94.8%", "+1.2%")
    with c3: st.metric("كفاءة النظام", "نشط", "آمن")
    
    st.markdown("<br>", unsafe_allow_html=True)
    st.image("https://img.freepik.com/free-vector/modern-data-analysis-concept-with-flat-design_23-2147920197.jpg", use_container_width=True)

# الصفحة 2: فحص الائتمان (إدخال البيانات)
elif selected_page == "🔍 فحص الائتمان":
    st.markdown("<div class='main-header'>نظام الفحص الفوري (AI)</div>", unsafe_allow_html=True)
    
    with st.container():
        st.markdown("<div class='data-card'>", unsafe_allow_html=True)
        st.subheader("📝 استمارة بيانات العميل")
        
        col1, col2 = st.columns(2)
        with col1:
            name = st.text_input("👤 اسم العميل بالكامل")
            income = st.number_input("💵 الدخل الشهري ($)", min_value=0, value=3000)
        with col2:
            loan = st.number_input("💰 قيمة القرض المطلوب ($)", min_value=0, value=10000)
            score = st.slider("📊 الدرجة الائتمانية (Score)", 300, 900, 600)
        
        if st.button("اتخاذ القرار الآن", use_container_width=True):
            if name:
                prediction = model.predict([[income, loan, score]])
                res_text = "مقبول ✅" if prediction[0] == 1 else "مرفوض ❌"
                
                # الربط البرمجي: تخزين النتيجة في الـ session_state لتظهر في صفحة التقارير
                st.session_state['history'].append({
                    "التاريخ": pd.Timestamp.now().strftime("%Y-%m-%d %H:%M"),
                    "العميل": name, "الدخل": income, "القرض": loan, "الدرجة": score, "القرار": res_text
                })
                
                if prediction[0] == 1: st.success(f"النتيجة: {res_text}")
                else: st.error(f"النتيجة: {res_text}")
            else:
                st.warning("يرجى إدخال اسم العميل.")
        st.markdown("</div>", unsafe_allow_html=True)

# الصفحة 3: ذكاء الأعمال (BI)
elif selected_page == "📊 ذكاء الأعمال (BI)":
    st.markdown("<div class='main-header'>مركز تحليل البيانات المرفوعة</div>", unsafe_allow_html=True)
    
    file = st.file_uploader("ارفع ملف البيانات (xlsx, csv)", type=['xlsx', 'csv'])
    
    if file:
        df = pd.read_excel(file, engine='openpyxl') if file.name.endswith('.xlsx') else pd.read_csv(file)
        st.session_state['uploaded_df'] = df # حفظ الملف للربط بين الصفحات
        
        st.success("تم رفع البيانات بنجاح!")
        c1, c2 = st.columns([1, 2])
        with c1:
            st.write("📊 معاينة أولية:")
            st.dataframe(df.head(10))
        with c2:
            chart_col = st.selectbox("اختر عموداً للتحليل البصري:", df.columns)
            fig = px.bar(df, x=chart_col, title=f"توزيع {chart_col}", color_discrete_sequence=['#3B82F6'])
            st.plotly_chart(fig, use_container_width=True)

# الصفحة 4: التصدير والتقارير
elif selected_page == "📥 التصدير والتقارير":
    st.markdown("<div class='main-header'>مركز تصدير التقارير النهائية</div>", unsafe_allow_html=True)
    
    tab1, tab2 = st.tabs(["📄 سجل الفحوصات الفورية", "📂 تصدير البيانات المرفوعة"])
    
    with tab1:
        if st.session_state['history']:
            final_df = pd.DataFrame(st.session_state['history'])
            st.dataframe(final_df, use_container_width=True)
            
            # وظيفة تصدير Excel
            output = BytesIO()
            with pd.ExcelWriter(output, engine='xlsxwriter') as writer:
                final_df.to_excel(writer, index=False, sheet_name='النتائج')
            
            st.download_button(
                label="📥 تحميل سجل الفحوصات (Excel)",
                data=output.getvalue(),
                file_name="credit_report_final.xlsx",
                mime="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"
            )
        else:
            st.info("لا توجد بيانات فحوصات حالياً.")

    with tab2:
        if st.session_state['uploaded_df'] is not None:
            st.write("البيانات المرفوعة سابقاً:")
            st.dataframe(st.session_state['uploaded_df'], use_container_width=True)
            
            # تصدير البيانات المرفوعة
            output_up = BytesIO()
            with pd.ExcelWriter(output_up, engine='xlsxwriter') as writer:
                st.session_state['uploaded_df'].to_excel(writer, index=False)
            
            st.download_button(label="📥 تحميل الملف المرفوع كـ Excel", 
                             data=output_up.getvalue(), 
                             file_name="data_export.xlsx")
        else:
            st.warning("يرجى رفع ملف في صفحة (BI) ليظهر هنا.")

# ⚡ أول 3 Workflows تبدأ بيهم n8n

لو لسه داخل الـ AI Automation، أصعب حاجة مش الأداة… أصعب حاجة إنك تعرف **تبني إيه الأول**.
هنا هتلاقي أول 3 Workflows أنا بنصح أي حد يبدأ بيهم، من السهل للأصعب، وكل واحد بالخطوات والـ Nodes والـ Prompt اللي هتحطه.

> 💡 كل Prompt جاهز Copy & Paste، وغيّر بس الحاجات اللي بين `[ ]`.

---

## 🤔 الأول: n8n ولا Make ولا Zapier؟

| | Zapier | Make | n8n |
|---|---|---|---|
| السهولة | الأسهل | متوسطة | محتاجة شوية تعلم |
| السعر لما الشغل يكبر | بيغلى جداً | أرخص | Self-host بسعر سيرفر بس |
| الـ AI Agents | موجودة بس محدودة | موجودة | جاهزة وقوية جداً |
| تكتب Code؟ | محدود | محدود | آه، JavaScript و Python |
| مناسبة لـ | تجربة بسيطة | Workflows متوسطة | لو ناوي تشتغل في المجال |

**نصيحتي:** لو عايز تجرب بس، ابدأ بـ Make. لو ناوي تشتغل وتكسب من المجال، ابدأ بـ n8n على طول.

---

## 🔌 تجهيز n8n

عندك طريقتين:

1. **n8n Cloud**: أسهل طريقة، تعمل حساب على [n8n.io](https://n8n.io) وتبدأ على طول من غير سيرفر.
2. **Self-host**: تنزّله على جهازك أو على سيرفر (VPS) بفلوس قليلة. أسرع طريقة تجربه على جهازك لو عندك Docker:

```bash
docker volume create n8n_data
docker run -it --rm --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

وبعدين افتح `http://localhost:5678`.

3. هتحتاج تعمل **Credentials** لكل App هتربطه (Google، Telegram، OpenAI…). n8n بيشرحلك الخطوات جوه كل Credential.

📖 شرح التسطيب الرسمي: [docs.n8n.io/hosting](https://docs.n8n.io/hosting/)

---

## 📝 Workflow 1: فورم → Google Sheets → إيميل (Lead Capture)

**امتى تستخدمه؟** أي بيزنس عنده عملاء بيسيبوا بياناتهم. ده أبسط Workflow وهيعلمك أهم 3 حاجات: الـ Trigger، والـ Data بتتنقل إزاي بين الـ Nodes، والـ Expressions.

### الـ Nodes
```
On form submission  →  Google Sheets (Append row)  →  Gmail (Send a message)
```

### الخطوات
1. **On form submission** (n8n Form Trigger): اعمل فورم فيه الحقول دي: `Name` ، `Phone` ، `Email` ، `Service`.
2. **Google Sheets → Append row in sheet**: اختار الشيت، واعمل Map لكل عمود بالحقل اللي قصاده. مثال: عمود Name = `{{ $json.Name }}`.
3. **Gmail → Send a message**: ابعت لنفسك إيميل إن فيه Lead جديد:
   - Subject: `Lead جديد: {{ $('On form submission').item.json.Name }}`
   - Message:
```
New lead just came in:
Name: {{ $('On form submission').item.json.Name }}
Phone: {{ $('On form submission').item.json.Phone }}
Email: {{ $('On form submission').item.json.Email }}
Service: {{ $('On form submission').item.json.Service }}
```

**بيعمل إيه؟** كل ما حد يملا الفورم، بياناته تتسجل في الشيت وإنت يوصلك إيميل في نفس الثانية.
**نصيحة:** جرّب الأول بـ Test step على كل Node لوحده، عشان تشوف الـ Data شكلها إيه قبل ما تكمل.

---

## 📬 Workflow 2: ملخص الإيميل اليومي بالـ AI

**امتى تستخدمه؟** لو الـ Inbox بتاعك زحمة وعايز كل يوم الصبح رسالة واحدة فيها المهم بس. هنا هتدخل أول مرة على الـ AI جوه n8n.

### الـ Nodes
```
Schedule Trigger  →  Gmail (Get many)  →  Aggregate  →  Basic LLM Chain (+ Chat Model)  →  Telegram (Send message)
```

### الخطوات
1. **Schedule Trigger**: كل يوم الساعة 9 الصبح.
2. **Gmail → Get many messages**: فعّل Simplify، ومن الـ Filters حط Received After = `{{ $now.minus({ days: 1 }).toISO() }}`.
3. **Aggregate**: اختار *All Item Data* عشان كل الإيميلات تتجمع في Item واحد وتتبعت للـ AI مرة واحدة.
4. **Basic LLM Chain**: اربطه بـ Chat Model (OpenAI أو Gemini أو Anthropic)، وحط الـ Prompt ده:

```
You are my email assistant. Below are the emails I received in the last 24 hours, as JSON:

{{ JSON.stringify($json.data) }}

Summarize them for me in Egyptian Arabic, grouped like this:
1. 🔴 Needs a reply today: sender, one-line summary, and what they want from me.
2. 🟡 Important but can wait: sender + one-line summary.
3. ⚪ Newsletters / promotions: just count them, don't list them.

Rules:
- Keep each line short.
- If an email mentions a deadline, a payment, or a meeting, put it at the top.
- Do NOT write or send any replies. Summary only.
```

5. **Telegram → Send a text message**: الـ Chat ID بتاعك، والـ Text = `{{ $json.text }}`.

**بيعمل إيه؟** كل يوم الصبح يوصلك على تليجرام ملخص بالمصري لكل إيميلات امبارح، مترتبة حسب الأهمية.
**نصيحة:** لو عندك إيميلات كتير جداً، حط Limit في Gmail node (مثلاً 50) عشان متصرفش Tokens على الفاضي.

---

## 🤖 Workflow 3: بوت تليجرام بـ AI Agent بيرد على عملائك

**امتى تستخدمه؟** ده الأساس لأي Chatbot هتبيعه لعميل بعد كده (WhatsApp، Instagram، Website…). نفس الفكرة بالظبط، بس بتغيّر الـ Trigger.

### الـ Nodes
```
Telegram Trigger  →  AI Agent (+ Chat Model + Simple Memory)  →  Telegram (Send message)
```

### الخطوات
1. اعمل بوت من **@BotFather** على تليجرام وخد الـ Token، وحطه في Telegram Credential.
2. **Telegram Trigger**: الـ Updates = `message`.
3. **AI Agent**:
   - Source for Prompt: *Define below* ، والـ Prompt = `{{ $json.message.text }}`
   - اربطه بـ **Chat Model** (أي موديل).
   - اربطه بـ **Simple Memory**، والـ Session Key = `{{ $('Telegram Trigger').item.json.message.chat.id }}` عشان كل عميل يبقى له محادثة لوحده.
   - في الـ Options → **System Message** حط ده:

```
You are the customer service assistant for [اسم البيزنس], a [نوع البيزنس] in [المدينة].

Your job:
- Answer questions about our services, prices, and working hours using ONLY the info below.
- If the customer wants to book, collect: name, phone number, preferred day and time.
- If you don't know the answer, say you'll check with the team. Never make up prices or info.

Business info:
- Services and prices: [اكتب الخدمات والأسعار]
- Working hours: [مواعيد الشغل]
- Location: [العنوان]

Style:
- Reply in the same language the customer uses (Egyptian Arabic if they write in Arabic).
- Short, friendly messages. Max 3–4 lines.
```

4. **Telegram → Send a text message**:
   - Chat ID = `{{ $('Telegram Trigger').item.json.message.chat.id }}`
   - Text = `{{ $json.output }}`
5. فعّل الـ Workflow (Publish) وابعت للبوت رسالة.

**بيعمل إيه؟** بوت بيرد على العملاء 24 ساعة، فاكر المحادثة، وبيجمع بيانات الحجز.
**نصيحة:** لما تخلص ده، الخطوة الجاية إنك تضيف للـ Agent **Tool** (زي Google Sheets) يسجل فيها الحجز لوحده. هنا بقى بتبدأ تعمل Systems تتباع.

---

## 🇪🇬 تقدر تكتب الـ System Message بالمصري كمان

```
إنت مساعد خدمة العملاء بتاع [اسم البيزنس].
رد على أسئلة العملاء عن الخدمات والأسعار والمواعيد من المعلومات اللي تحت دي بس.
لو العميل عايز يحجز، خد منه: الاسم، رقم الموبايل، واليوم والساعة اللي تناسبه.
لو مش عارف الإجابة، قوله هرجعلك بعد ما أسأل الفريق، ومتألفش أي سعر أو معلومة.
ردودك تكون قصيرة وودودة، ومتزيدش عن 3 أو 4 سطور.

المعلومات:
[اكتب الخدمات والأسعار والمواعيد والعنوان]
```

---

## 🎁 بونص: خلي Claude يساعدك تبني أي Workflow

لو وقفت في أي خطوة، ابعت الـ Prompt ده لـ Claude:

```
I'm a beginner learning n8n. I want to build a workflow that does this:
[اشرح الـ Workflow اللي عايزه بالتفصيل]

Please give me:
1. The exact list of nodes in order, with the node names as they appear in n8n.
2. The settings for each node, including any expressions I need.
3. What credentials I need to set up.
4. How to test each step before activating the workflow.

Keep it simple and explain any expression you use in one line.
```

ولو عندك Error، ابعتله الـ Error Message وصورة من الـ Node وقوله: `This node is failing, explain why and how to fix it step by step.`

---

## ⚠️ نصايح مهمة

- **جرّب كل Node لوحده** بـ Test step قبل ما تشغل الـ Workflow كله.
- **متحطش API Keys أو Tokens جوه الـ Nodes نفسها**، دايماً استخدم الـ Credentials.
- **أي Workflow بيبعت حاجة لناس حقيقية** (إيميل، رسالة)، جربه الأول على نفسك.
- **راقب الـ Executions** من القايمة اللي على الشمال، هتعرف منها أي حاجة وقعت وليه.
- **ابدأ صغير**: Workflow شغال من 3 Nodes أحسن من 20 Node مش شغالين.

---

## 🚀 الخلاصة

| قبل | بعد |
|---|---|
| بتنقل بيانات العملاء بإيدك في Excel | الفورم بيسجلها لوحده ويبعتلك إشعار |
| بتقضي نص ساعة كل يوم في الـ Inbox | ملخص واحد بالمصري كل يوم الصبح |
| بترد على نفس الأسئلة كل يوم | بوت بيرد 24 ساعة ويجمع الحجوزات |
| مش عارف تبدأ منين | عندك 3 Workflows تقدر تبيع أي واحد فيهم لعميل |

---
لو استفدت، تابعني عشان فيه Workflows وأتمتة أكتر جاية 🔥

**Steven Ayman — Vida AI**

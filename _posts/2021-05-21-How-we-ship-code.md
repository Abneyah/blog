---
layout: post
title: How we ship code
---
نسعى في [أبنية](https://abneyah.com/) إلى جعل تجربة بناء و إطلاق برامجنا سريعة وسلسة من خلال استخدام تقنيات وطرق تمكننا من أتمتة بناء وتطوير برامجنا بأسرع طريقة وإلغاء أي تدخل بشري. سأذكر في هذه المدونة كيفية استخدامنا لخدمات أمازون السحابية [(AWS)](https://aws.amazon.com/) في بناء code pipeline لأتمتة تحديث موقع وتطبيق منصة أبنية. على الرغم من أن التفاصيل التي سوف أذكرها خاصة بتحديث الموقع، إلا أن طريقة عمل تحديث التطبيق مشابه إلى حد كبير لطريقة تحديث الموقع وقد نذكر طريقة تحديثنا للتطبيق في مدونة لاحقة.

## Architectural Diagram
<img src="https://blog.abneyah.com/public/img/cicd.png" alt="Abneyah">

## AWS CodePipeline
توفر خدمات أمازون للخدمات السحابية خدمة [AWS CodePipeline](https://aws.amazon.com/codepipeline/) والتي نستخدمها لإدارة كامل دورة عملية بناء وتحديث البرامج حيث تقوم هذه الخدمة بالتأكد من أن كامل متطلبات العمل تتم وفق المراحل - Stages- التي نحددها. تنفيذ المراحل يكون بشكل تسلسي وبالإمكان تنفيذ أكثر من عمل في آن واحد خلال مرحلة واحدة إن أراد المستخدم ذلك. من المزايا الرائعة التي تقدمها خدمة AWS CodePipelin الدعم لعدد من الحلول والبرامج المعروفة بخدماتها في مجال بناء وإصدار البرامج. إذا لا تقيد AWS المستخدمين باستخدام خدماتها فقط وإنما بإمكانهم استخدام بعض من الحلول والبرامج التي لها شعبية واسعة مثل [GitHub](https://github.com/) و [Bitbucket](https://bitbucket.org/product) و [Jenkins](https://www.jenkins.io/).

<img src="https://blog.abneyah.com/public/img/codepipeline.png" alt="Abneyah">

### Version Control
جميع ملفات البرمجة التي نبنيها تكون مخزنة في GitHub. على الرغم من أن أمازون للخدمات السحابية توفر خدمة مشابهة لـ GitHub وهي [AWS CodeCommit](https://aws.amazon.com/codecommit/) إلى أننا فضلنا استخدام GitHub لعدد من الأسباب من أهمها استخدامنا لـ [GitHub Projects](https://github.com/features/project-management/) لإدارة الجانب التقني لمشروع أبنية. فبدلاً من استخدام برامج إدارة مشاريع أخرى مثل [Atlassian Jira](https://www.atlassian.com/software/jira), [Trello](https://trello.com/en), [Asana](https://asana.com/) أو غيرها من برامج إدارة المشاريع. فإننا نستخدم GitHub Projects لتقسيم مشروع أبنية إلى مهام وإسناد المهمات ومراجعتها بيننا. وبخلاف AWS CodeCommit الذي يقتصر استخدامه على المتصفح أو CLI, توفر GitHub نسخة للبرنامج على الأجهزة الذكية و التي تمكننا من متابعة إدارة المشروع من أي مكان.

### AWS CodeBuild
يبدأ AWS CodePipeline بالعمل بواسطة [webhook](https://docs.github.com/en/developers/webhooks-and-events/webhooks/about-webhooks) ترسل له عن طريق GitHub في كل مرة نقوم بعمل تغييرات جديدة للبرامج. عند تشغيل الـ pipeline الخاص بالبرنامج الذي تم تحديثه، تقوم الخدمة بتشغيل خدمة [AWS CodeBuild](https://aws.amazon.com/codebuild/) المسؤولة عن أتمتة بناء الأكواد. تقوم خدمة AWS CodeBuild بتشغيل [container](https://www.docker.com/resources/what-container) يقوم بتحميل الأكواد الموجودة في GitHub وبنائها ثم حفظ ناتج البناء في [S3 Bucket](https://aws.amazon.com/s3/) كما هو موضح في المخطط.  خطوات بناء البرنامج من خلال الأكواد تكون موضحة في ملف [buildspec.yaml](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html). يمكننا هذا الملف من تحديد البرامج المطلوبة لبناء البرنامج مثل أوامر ما قبل البناء، أثناء البناء أو بعد البناء بالإضافة لعدد من الخيارات التي قد يحتاجها المطور لبناء البرنامج.

### AWS Identity and Access Management (IAM)


لكي يتمكن أي شخص أو خدمة باستخدام خدمة أخرى يجب أن تكون له أو لها صلاحية لاستخدام الخدمة ويتم ذلك من خلال خدمة [IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html). هناك نوعين أساسيين من المستخدمين الذين توفرهم IAM للأشخاص أو الخدمات لغرض استخدام خدمات AWS وهما IAM User وIAM User. IAM Role نوع من المستخدمين الذي يتم استخدام صلاحياته بواسطة كلمات سرية Access and Secret keys طويلة الأمد. IAM Role هي وظيفة بإمكان المستخدم أو الخدمة تقمصها لاستخدام الخدمات ويتم ذلك عن طريق قيام AWS بعمل Access key, Secret key, Session Token مؤقتاً نيابة عنا لأداء الخدمة التي نرغب بتنفيذها وتعطيل المفاتيح بعد الانتهاء. استخدام IAM Role بدلاً من IAM User يعتبر الخيار الأسلم والآمن في كل الأوقات متى كان ذلك ممكنا. يتم تحديد صلاحيات الـ IAM User أو IAM Role باستخدام IAM Policies. المثال التالي هو نموذج لـ IAM Policy يتم إضافته لـIAM Role  أو IAM User لتحديد صلاحياته. المهم في هذا المثال ٣ أشياء أساسية وهي Effect, Action, Resource

<img src="https://blog.abneyah.com/public/img/policy.png" alt="Policy">

*   Action: تحديد نوع الفعل و هنا نحدد أن الهدف هو تشغيل codebuild بواسطة StartBuild
*   Effect: نوع الصلاحية إما إعطاء صلاحية بـAllow أو تعطيل صلاحية بـDeny
*   Resource:  الـresource التي يطبق عليه الصلاحية. بإمكاننا تحديد resource واحد أو مجموعة باستخدام 
array أو regex


لكل resource في AWS اسم خاص يتفرد به ويميزه عن أي resource في كل حسابات AWS الأخرى. يطلق على هذا الإسم Amazon Resource Name أو [ARN](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html) ويحتوي بشكل أساسي اسم المنطقة الجغرافية التي يتواجد فيها الـ resource. رقم حساب و اسم الـresource نفسه. 



لكي يقوم AWS CodeBuild بنقل البرنامج الذي تم الانتهاء من إلى S3 Bucket, قمنا بإعطائه IAM Role ذو صلاحيات لنقل البرنامج إلى S3 Bucket. S3 هي الخدمة التي نستخدمها لإستضافة موقع أبنية و بإمكانكم التعرف على تفاصيل استضافتنا لموقع أبنية من خلال هذا [الرابط](https://blog.abneyah.com/2021/05/20/Engineering-Abneyah-website/)

### AWS Lambda & Abneyah Bot
نستخدم Slack للتواصل بيننا. فالإضافة إلى سهولة استخدامه، يدعم Slack عدداً من البرامج التي نستخدمها لإدارة المنصة ومواقعنا الإلكترونية. قمنا بإنشاء Abneyah Bot فيSlack  لإرسال تنبيهات لنا من ضمنها تنبيهنا حال انتهاء AWS CodeBuild من تحديث مواقعنا الإلكترونية أو التطبيق. نرسل طلب Slack webhook باستخدام [AWS Lambda Function](https://aws.amazon.com/lambda/) و التي تمكننا من تنفيذ  - invoke -الطلب دون الحاجة لاستخدام خوادم خاصة بنا. إذ تقوم AWS بتفيذ أكواد البرمجة باستخدام خوادمها الخاصة ونقوم بدفع تكلفة تشغيل الـFunction لحين انتهائها من تنفيذ الطلب. لكي نقوم بعمل – invocation – للـ Lambda Function, أضفنا IAM policy لـIAM Role الخاص بالـ AWS CodePipeline. 

### حفظ المفاتيح السرية
يتطلب استخدام Slack webhook إلى إصدار Slack webhook URL خاص بنا لإرسال الرسائل والتنبيهات. ولكي نتأكد من حفظه في مكان آمن، قمنا باستخدام خدمة [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html) لحفظ Slack webhook URL بطريقة مشفرة. توفر هذه الخدمة إمكانية حفظ الكلمات - strings - على هيئة key-value pair ويستطيع المستخدم تحديد ما إذا كان يريد حفظ الـكلمات مشفرة أو غير مشفرة. تعتمد خدمة AWS Systems Manager Parameter Store في تشفير البيانات على خدمة [AWS Key-Management Service](https://aws.amazon.com/kms/) المسؤولة عن إصدار وإدارة مفاتيح التشفير والتي تدعم تشفير عدد كبير من خدمات أمازون السحابية.

خدمة AWS Systems Manager Parameter Store ليست الخدمة الوحيدة التي تقدمها AWS لحماية الكلمات السرية الخاصة بالمستخدمين إذ أن خدمة [AWS Secret Manager](https://aws.amazon.com/secrets-manager/) تقوم بنفس الوظيفة بالإضافة إلى وظائف متخصصة أكثر مثل إنشاء كلمات سرية بالنيابة عن المستخدم و أتمتة تدوير الكلمات السرية والتي قد تكون من متطلبات الإمتثال الأمني   - Security Compliance – بالقيام بتدوير الكلمات السرية بشكل دوري. تدعم هذه الخدمة أتمتة تدوير الكلمات السرية لبعض من قواعد البيانات التي توفرها AWS مثل AWS RDS, AWS RedShift.  

فضلنا استخدام AWS Systems Manager Parameter Store على AWS Secret Manager لسبب إقتصادي بحت حيث AWS Systems Manager Parameter Store خدمة مجانية في المقابل استخدام خدمة  AWS Secret Manager  تكون بمقابل بسيط جداً لكل كلمة سرية تديرها AWS.



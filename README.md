# cv_eye_detection
 Передо мной стояла задача классификации по определению положения глаза (закрыт или открыт) по обучающей выборке. Датасет называется EyesDataset, который включает в себя 4000 неразмеченных фотографий, каждая из которых одноканальная, размером 24 на 24 пикселя. 
Основанная сложность нашей задачи в том, что выборка оказалось неразмеченной. Был велик соблазн вручную соотнести метки с каждой фотографией, но мне показалось, что тогда это будет слишком легким и заурядным решением проблемы, ведь скачать готовую сеточку / построить архитектуру с нуля на полностью готовых данных банально и в наше время этим уже никого не удивишь.

В процессе исследования я нашел два интересных подхода к решению проблемы неразмеченных данных. Первый вариант – это полуконтролируемое обучение. Углубившись, более внимательно я понял, что так называют подход в кластеризации, который обобщает данные в группы отталкиваясь от уже размеченных данных. То есть соединяя в себе преимущества supervised и unsupervised learning. Второй вариант – маркировать только те фотографии, в которых модель сильно уверена. 
Для осуществления этих подходов все равно, необходима часть размеченных данных. Таким образом я решил вручную разметить 2000 фотографий, чтобы половина из них пошла на валидацию моей модели, а вторая часть участвовала в обучении.
Вторая большая проблема, которая стояла перед мной, этой малый размер данных для обучения, поэтому я решил применить различные техники аугментации, чтобы увеличить бесплатно размер размеченных данных (картинка 1). Путем комбинации методов аугментации мне получилось увеличить свой датасет почти в 35 раз. После я обучил resnet15 на своих данных и получил accuracy на test примерно 96.59.

В процессе исследования я выяснил, что в сете встречаются много аномалий, от которых хотелось бы избавиться, желательно, не ручным способом. Таким образом, пришлось затронуть тему обучение без учителя. Сначала применил алгоритм кластеризации dbscan для поиска аномалий. После решил продолжить поиск выбросов путем оценки контрастности фотографий. В итоге нашел порядка 15 пример, которые с трудом напоминали бы глаза. Пример выбросов видно на картинке 2. Подчистив немного отложенную выборку, решил перейти к полуконтролируемому обучению.

                                
         <img width="148" alt="image" src="https://user-images.githubusercontent.com/50549021/164486817-97007846-84aa-4647-a083-337f333613c6.png">

         
Пройдя через небольшие тернии подготовки данных, я наконец, приблизился к первому варианту решению главной проблемы. Sklearn предлагал несколько вариантов решения,  я остановился на LabelSpreading. Так как алгоритм не работает с фотографиями, то мои двухмерные матрицы пришлось конкатенировать в одномерные вектора с последующим сжатием через PCA. Результаты меня обрадовали, так как данный алгоритм смог уловить закономерность на отложенной выборке под 98%. Следующем шагом, я решил посмотреть, сколько ошибок на test выборке я получу если оставлю только те результаты, в которых модель уверена выше некоторого порога. Тем самым, рассчитывая получить такой же процент верных ответов на тех данных, которые я еще не размечал. После окончания всей процедуры я добавил 182 новые фотографии к своему тренировочному датасету и решил до обучить модель. К моему разочарованию я смог повысить качество на test только на до 96.80, но качество на train у меня сильно просело, с 0.98 до 0.93.
Сразу после этого решил применить второй подход. То есть добавить в свой train ту часть данных, в которых модель уверена больше всего. Так же, как и на прошлом шаге, найдя наилучший порог уверенности по тестовой выборке, применил его на неразмеченных данных. После трехкратного повторения этой процедуры решил остановиться, добавив 450 новых, размеченных моделью, данных в свой train. До обучив модель я понизил accuracy на тестовой выборке до 0.96, но повысил результаты на train.
Финальный roc_auc_score вышел под 0.973, а EER оказался под 0.0309. Довольно неплохо, учитывая, что размер test равен 1000 элементов. Надеюсь, что и на других отложенных выборках результаты будут столь высокие. 
Подводя итоги, хочу сказать, что, данные процедуры я повторял несколько раз и каждый раз результаты менялись несущественно. Первый вариант не дал мне существенных изменений, а второй немного переобучил мою модель. Большой пользы ssl в данном проекте мне не принес, надеюсь, что в будущем я смогу переосмыслить полученные результаты и вывести пользу из ssl. В целом работа проектом показалась мне интересным шагом, в первую очередь, в развитии самого себя в сфере распознавания лиц. Хотелось бы и дальше продолжать работу в этом направлении.


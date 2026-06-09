
# 📊 Схема бизнес-процесса: АРМ по работе с СИ
```mermaid
graph TD
    subgraph POOL_АРМ [ПУЛ: АРМ по работе с Приборами Измерений]
        
        subgraph LANE_M [🧑‍🔧 Дорожка: Метролог]
            Start((Начало работы АРМ)) --> OpenList[Открыть окно «Экземпляры СИ»]
            OpenList --> ApplyFilter[Применить фильтр: ЮЛ → Подразделение → Ответственный]
            ApplyFilter --> ViewList[Отображение перечня СИ + автосчетчик]
            ViewList --> ActionChoice{Выбор действия}
            
            ActionChoice -->|Создать| CreateCard[Нажать «Создать карточку СИ»]
            CreateCard --> FillData[Заполнить поля карточки]
            FillData --> CheckArshin{Тип СИ утверждён?}
            CheckArshin -->|Да| FetchArshin[Авто-загрузка из Аршина]
            CheckArshin -->|Нет| ManualFill[Ручной ввод]
            FetchArshin --> Validate{Валидация полей}
            ManualFill --> Validate
            Validate -->|Ошибка| Highlight[Подсветка пустых полей + Сообщение «внесите обязательную информацию»]
            Highlight --> FillData
            Validate -->|ОК| AutoInv[Авто-генерация инв.№ формата ццц-б]
            AutoInv --> SaveCard[Сохранить карточку]
            
            ActionChoice -->|МК/Изменения| FindSI[Поиск СИ по №/наименованию/статусу]
            FindSI --> SelectRow[Клик по строке → История МК]
            SelectRow --> OpenMKForm[Нажать «Внести данные о МК»]
            OpenMKForm --> FillMK[Дата, Результат, Исполнитель, Документы]
            FillMK --> CalcNext{Результат МК «Годен»?}
            CalcNext -->|Да| AutoDate[Авто-расчёт даты след. МК]
            CalcNext -->|Нет| ClearNext[Блокировка/очистка даты след. МК]
            AutoDate --> SaveMK[Сохранить данные МК]
            ClearNext --> SaveMK
            
            ActionChoice -->|Статус/Логистика| ChangeStatus[Изменение статуса через выпадающее меню]
            ChangeStatus --> SetPrep[Статус: Подготовка к МК]
            ChangeStatus --> SetMKStat[Статус: В поверке/калибровке/аттестации]
            ChangeStatus --> SetWh[Статус: На складе / В ОФ]
            ChangeStatus --> SetOp[Статус: В эксплуатации]
            ChangeStatus --> SetWO[Статус: Списан]
            SetWO --> MoveArchive[Перенос карточки в архив списанных СИ]
        end

        subgraph LANE_S [⚙️ Дорожка: Система (Автоматизация)]
            SaveCard -.->|Триггер 18:00| NotifyReg[📩 Рассылка: Постановка на учет (Форма 1)]
            SetPrep -.->|Триггер 18:00| NotifyPrep[📩 Рассылка: Сдача СИ (Форма 5)]
            SetMKStat -.->|Триггер 18:00| NotifyReady[📩 Рассылка: Готовность СИ (Форма 4)]
            SetWh -.->|Триггер 18:00| NotifyWh[📩 Рассылка: Передача на склад (Форма 7)]
            SetOp -.->|Триггер 18:00| NotifyOp[📩 Рассылка: Получение СИ (Форма 8)]
            
            Timer1M((⏱️ Таймер: -1 мес до МК)) --> SendMonth[📩 Уведомление о сдаче (Форма 2)]
            SendMonth -.->|⏱️ Таймер: 2 недели| CheckStat{Статус изменился?}
            CheckStat -->|Нет| ReNotify[📩 Повторное уведомление]
            CheckStat -->|Да| StopNotif((Конец ветки))
            
            TimerOver((⏱️ Таймер: Просрочка МК)) --> AlertEscalate[📩 Уведомление «ВНИМАНИЕ!» (Форма 3) + Эскалация руководству]
        end

        subgraph LANE_A [🛡️ Дорожка: Администратор]
            DictManage[Заполнение/редактирование справочников]
            DictManage -.->|Обновление метаданных| UpdateMenus[Выпадающие списки в АРМ]
        end

        subgraph LANE_U [👥 Дорожка: Начальник/Мастер/Контроллер]
            ViewListU[Просмотр перечней СИ]
            ApplyFilterU[Фильтрация по ЮЛ/Подразделению/Ответственному]
            ApplyFilterU --> ExportExcelU[📤 Выгрузка в Excel]
            ApplyFilterU --> GenReportU[📊 Формирование отчетов/графиков]
            GenReportU --> Template[⚙️ Настройка шаблона: подписанты, столбцы, оформление]
            Template --> ExportReport[📤 Экспорт отчета/графика]
            
            MasterNI[Мастер НИЗМК: Отметка получения со склада]
            MasterNI --> ChangeStatusMaster[Статус: В эксплуатации + Выбор участка/ответственного]
            ChangeStatusMaster -.->|Триггер 18:00| NotifyMaster[📩 Уведомление о получении (Форма 6)]
        end
    end
```

    %% Внешние системы и хранилища
    Arshin[(🌐 База Аршин)] -.->|REST/Интеграция| FetchArshin
    Excel[(📁 Файлы Excel)] -.->|Экспорт| ExportExcelU
    ArchiveDB[(💾 Архив списанных СИ)] -.->|Хранение статистики| MoveArchive

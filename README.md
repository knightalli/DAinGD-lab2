# DAinGD-lab2
# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил(а):
- Миронова Наталья Андреевна
- РИ220930
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.


## Задание 1
### Выберите одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры. Выберите одну из игровых переменных в игре (ресурсы, внутри игровая валюта, здоровье персонажей и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.

Ход работы:
- Игра Eatventure - симулятор кафе, кликер.
- Вы играете за повара, который развивает свобственное кафе. Геймплей состоит из улучшения станций с едой, найма работников, общих улучшений (скорости покупателей, количества гемов с рекламы и т.д.). Задача состоит в том, чтобы собрать как можно больше денег и быстрее пройти города.
- Я выбрала чаевые
- Из игровых переменных я выбрала чаевые от покупателей. После получения заказа покупатель может оставить чаевые по стоимости половины средней цены продаж, полной цены продаж или вообще не оставляют (0%, 50%, 100%).

![задание](https://github.com/knightalli/DAinGD-lab2/assets/127225486/e4fe9848-ed58-40ad-8c56-d563812c4af9)


## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в выбранной игре (в качестве таких переменных может выступать игровая валюта, ресурсы, здоровье и т.д.). Средствами google-sheets визуализируйте данные в google-таблице (постройте график, диаграмму и пр.) для наглядного представления выбранной игровой величины.

Ход работы: 
Я настроила код в Jupyter Notebook под выбранную мной переменную. Выводила полученные (или нет) чаевые.

![image](https://github.com/knightalli/DAinGD-lab2/assets/127225486/6a8b6ecb-d9b3-4632-b296-2a9d3405d060)


```py

import gspread
import numpy as np
gc = gspread.service_account(filename='unitydatasciencemir-dc3efe934fe4.json')
sh = gc.open("UnityWorkshop2")
tipsPercent = np.random.randint(0, 3, 11)
startPrice = 10
mon = list(range(1,11))
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = startPrice*tipsPercent[i-1] - startPrice   
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.',',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(tipsPercent[i-1]*50))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(tempInf)

```

## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

Ход работы:
Я настроила звуки в Unity так, чтобы "хороший" звук означал получение 100% чаевых, "средний" - 50%, "плохой" - 0.

![image](https://github.com/knightalli/DAinGD-lab2/assets/127225486/695cb6f6-67b1-4e0f-8392-35390224b957)

```cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class SoundScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string,float>();
    private bool statusStart = false;
    private int i = 1;
    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (!dataSet.ContainsKey("Mon_" + i.ToString())) return;

        if (dataSet["Mon_" + i.ToString()] == 10 & 
            statusStart == false & 
            i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] == 0 & 
            statusStart == false & 
            i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] == -10 &
            statusStart == false &
            i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest currentResp = 
            UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1ochG7_byyxdtne0ElPsv821pgUbEujbsXvaU2cvcgDg/values/A1%3AZ100?key=AIzaSyCkyWU1rLRpDtY3GjXL4KYdXczvC_YNJjM");
        yield return currentResp.SendWebRequest();
        string rawResp = currentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }

    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }

    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
}


```

## Выводы

Во время данной лабораторной работы я научилась заполнять Google Sheets с помощью Python. Также работать со звуками в Unity и методом StartCoroutine(). Я подключила необходимые сервисы по инструкции, переписала их под свою переменную, описав её до этого, заполнила таблицу и использовала эти данные в Unity для воспроизведения звуков при определенных условиях.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**

# tortoise-tts-rvc-voice-cloning-pipeline

## rvc voice cloning
- [Retrieval-based-Voice-Conversion-WebUI]([url](https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI/blob/main/docs/README.en.md))
- [The "AI COVER" tech they never tell you about (and how they work)]([url](https://www.youtube.com/watch?v=vhArHsfsLAQ&ab_channel=bycloud))
- [The BEST LOCAL AI Voice Cloning TTS Pipeline - Tortoise TTS + RVC]([url](https://www.youtube.com/watch?v=9wu6LSue_dU&list=PLknlHTKYxuNshtQQQ0uyfulwfWYRA6TGn&index=2&ab_channel=JarodsJourney))

### Model training

We start by running this [Google Colab]([url](https://colab.research.google.com/github/RVC-Project/Retrieval-based-Voice-Conversion-WebUI/blob/main/Retrieval_based_Voice_Conversion_WebUI.ipynb
)) that will launch a Gradio UI for us to pre-process our input audio, extract features from it, train a voice model, and use it for inference. Finally, we will convert this trained model into ONNX to use it outside this web application.

In the first step, we need an input audio file as training data. I have used WhatsApp audio files (1-2 min each, 5 minutes in total) but in this case we'll use a TED talk from YouTube. You could use [YouTube-DL]([url](https://github.com/ytdl-org/youtube-dl)https://github.com/ytdl-org/youtube-dl) in your pipeline but in case you don't want to, you can use online tools such as https://savetube.io, to download the audio from these videos.

Once we have the audio, we'll upload it to our Google Drive inside a folder that we'll name `rvc-dataset`. We should take the audio file (`.mp3`, `.ogg`, `.wav`, etc) and compress it into a `.zip` file that we'll then upload to our Drive.

In the Colab, we run the following cells
```
# @title 从谷歌云盘加载打包好的数据集到/content/dataset

# @markdown 数据集位置
DATASET ="/content/drive/MyDrive/rvc-dataset/milei.zip"  # @param {type:"string"}

!mkdir -p /content/dataset
!unzip -d /content/dataset -B {DATASET}
```

and 
```
# @title 重命名数据集中的重名文件
!ls -a /content/dataset/
!rename 's/(\w+)\.(\w+)~(\d*)/$1_$3.$2/' /content/dataset/*.*~*
```

This way, we are going to extract the `.zip` file we selected into `/content/dataset` in the local Colab folder.

Now we get to the next cell, where we launch the Gradio UI:
```
# @title 启动web
%cd /content/Retrieval-based-Voice-Conversion-WebUI
# %load_ext tensorboard
# %tensorboard --logdir /content/Retrieval-based-Voice-Conversion-WebUI/logs
!python3 infer-web.py --colab --pycmd python3
```

Now inside the Gradio UI, we go to `Train`, select `Input experiment name`, `Target sample rate`, `Input training folder path`, and click `Process data`. You should see something like this under `Export message`:
```
start preprocess
['trainset_preprocess_pipeline_print.py', '/content/dataset', '40000', '2', '/content/Retrieval-based-Voice-Conversion-WebUI/logs/milei', 'False']
/content/dataset/milei-voice.ogg->Suc.
end preprocess
```

![image](https://github.com/machinelearnear/tortoise-tts-rvc-voice-cloning-pipeline/assets/78419164/9f5323b1-f2e1-4250-a17b-1e0b822ffb39)

Now that we have finished this, we need to extract the **pitch** and extract the **voice features**. We do this by clicking `Feature extraction`. All results will be saved under `./logs` under a filename that looks like this `/content/Retrieval-based-Voice-Conversion-WebUI/logs/milei/trained_IVF1461_Flat_nprobe_1.index`.

![image](https://github.com/machinelearnear/tortoise-tts-rvc-voice-cloning-pipeline/assets/78419164/ada9afeb-4d2b-4046-8da5-171453d26010)

We are almost done now. We click `Train Feature Index` first and then `Train Model`. We leave all other options the same as in the picture.

![image](https://github.com/machinelearnear/tortoise-tts-rvc-voice-cloning-pipeline/assets/78419164/fcd4847e-5b96-4323-80a2-04e5efc5dd59)

Done! We've got our model now. It's saved under e.g. `/content/Retrieval-based-Voice-Conversion-WebUI/weights/milei.pth`

### Model inference

What we want to do now is use our model to convert a normal audiostream with someone's voice into a converted audiostream with our trained model's voice. We do this by going to the `Model inference` tab and selecting our model from the drop-down list (click `Refresh` if you don't see it).

![image](https://github.com/machinelearnear/tortoise-tts-rvc-voice-cloning-pipeline/assets/78419164/5a03e0de-ac04-43c3-a6d1-e34fb0d4524b)

Now select the path to the audio file you want to process (Google Drive) and your `Feature search database file path`. Finally, click `Convert`

![image](https://github.com/machinelearnear/tortoise-tts-rvc-voice-cloning-pipeline/assets/78419164/76f1dad9-98cd-43be-96be-e9395ef03949)


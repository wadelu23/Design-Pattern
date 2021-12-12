# Factory Pattern (工廠模式)

筆記程式碼範例來源

[The Factory Pattern in Python // Separate creation from use](https://www.youtube.com/watch?v=s_4ZrtQs8Do&list=PLC0nd42SBTaNuP4iB4L6SJlMaHE71FG6N&ab_channel=ArjanCodes)

[The code](https://github.com/ArjanCodes/2021-factory-pattern)

---

## 範例概述
根據使用者輸入的高低品質，獲得對應的Exporter處理檔案
```python
# 取得實體(改良前)
# export_quality( User input )
video_exporter: VideoExporter
audio_exporter: AudioExporter
if export_quality == "low":
    video_exporter = H264BPVideoExporter()
    audio_exporter = AACAudioExporter()
elif export_quality == "high":
    video_exporter = H264Hi422PVideoExporter()
    audio_exporter = AACAudioExporter()
else:
    # default: master quality
    video_exporter = LosslessVideoExporter()
    audio_exporter = WAVAudioExporter()
```

---

改以 ExporterFactory( abstract class ) 定義工廠

```python
class ExporterFactory(ABC):
    """
    Factory that represents a combination of video and audio codecs.
    The factory doesn't maintain any of the instances it creates.
    """

    @abstractmethod
    def get_video_exporter(self) -> VideoExporter:
        """Returns a new video exporter belonging to this factory."""

    @abstractmethod
    def get_audio_exporter(self) -> AudioExporter:
        """Returns a new audio exporter belonging to this factory."""


class FastExporter(ExporterFactory):
    """Factory aimed at providing a high speed, lower quality export."""

    def get_video_exporter(self) -> VideoExporter:
        return H264BPVideoExporter()

    def get_audio_exporter(self) -> AudioExporter:
        return AACAudioExporter()


class HighQualityExporter(ExporterFactory):
    """Factory aimed at providing a slower speed, high quality export."""

    def get_video_exporter(self) -> VideoExporter:
        return H264Hi422PVideoExporter()

    def get_audio_exporter(self) -> AudioExporter:
        return AACAudioExporter()

```

使用者指定高中低品質

獲得對應的工廠

如 class FastExporter( ExporterFactory )

```python
def read_factory() -> ExporterFactory:
    """Constructs an exporter factory based on the user's preference."""

    factories = {
        "low": FastExporter(),
        "high": HighQualityExporter(),
        "master": MasterQualityExporter(),
    }
    while True:
        export_quality = input("Enter desired output quality (low, high, master): ")
        if export_quality in factories:
            return factories[export_quality]
        print(f"Unknown output quality option: {export_quality}.")
```

由`get_video_exporter`,`get_audio_exporter`取得實體並使用

使用時不需知道該實體中詳細資料(如編碼方式、格式類型)

```python
def main(fac: ExporterFactory) -> None:
    """Main function."""

    # retrieve the exporters
    video_exporter = fac.get_video_exporter()
    audio_exporter = fac.get_audio_exporter()

    # prepare the export
    video_exporter.prepare_export("placeholder_for_video_data")
    audio_exporter.prepare_export("placeholder_for_audio_data")

    # do the export
    folder = pathlib.Path("/usr/tmp/video")
    video_exporter.do_export(folder)
    audio_exporter.do_export(folder)
```

符合 SOLID 原則的開放封閉原則

能夠擴展功能(如加一個第四種品質Exporter)

且擴展時不修改到原有的程式
# Python 실행 환경

Python의 실행 환경(가상환경, Jupyter, Anaconda, uv 등)에 대한 개념을 정리한 문서입니다.

## 실행 환경, Jupyter, Anaconda, uv

### VSCode에서 Python + Jupyter 구조

* VSCode에서:

  * Python 확장을 설치 → Python 인터프리터 선택.
  * Jupyter 확장을 설치 → 노트북 파일(`.ipynb`)에서 **커널(kernel)** 선택.
* 커널 = “어떤 Python 환경에서 코드를 실행할지”를 나타내는 실행 엔진.

  * 로컬 가상환경(venv, conda env 등),
  * 시스템 Python,
  * 특정 도커/원격 환경 등이 될 수 있다.

### “앱이 자체 파이썬 실행환경을 가진 경우”

* 특정 앱(예: 일부 데이터툴, IDE)은 자체 내장 Python을 가지고 올 수 있다.
* 이 경우:

  * 호스트에 Python을 따로 설치하지 않아도 **그 앱 안에서만** Python 코드 실행이 가능하다.
  * 다른 앱에서 재사용하려면 별도의 시스템/가상환경 Python이 필요하다.

### 가상환경과 버전

* 가상환경(venv/conda env)은:

  * 특정 Python 인터프리터 버전(예: 3.10)을 기반으로
  * 독립적인 site-packages 디렉토리를 갖는 구조.
* 호스트 Python을 지우더라도:

  * 가상환경이 호스트 인터프리터를 직접 가리키고 있었다면 깨질 수 있다.
  * 별도 바이너리로 포함된 형태라면 독립적으로 살아남을 수 있다(도구/설정에 따라 다름).

### Anaconda vs uv

* **Anaconda**

  * 데이터 과학/머신러닝 패키지를 통합 제공하는 배포판.
  * `conda` 명령으로 파이썬 버전 + 패키지 + 가상환경을 통합 관리.
* **uv**

  * Rust로 구현된 차세대 Python 패키지/환경 관리 도구.
  * pip, venv, pipx, poetry 역할을 통합하려는 지향.
  * 빠른 설치 속도, 캐시 효율, 의존성 관리 개선을 목표로 한다.

# llamafile

[![ci status](https://github.com/Mozilla-Ocho/llamafile/actions/workflows/ci.yml/badge.svg)](https://github.com/Mozilla-Ocho/llamafile/actions/workflows/ci.yml)

[![status](https://dcbadge.vercel.app/api/server/YuMNeuKStr)](https://discord.gg/YuMNeuKStr)

![llamafile](llamafile/llamafile-640x640.png)

**llamafile pozwala na dystrybucję i uruchamianie LLM za pomocą pojedynczego pliku.** ([ogłoszenie na blogu](https://hacks.mozilla.org/2023/11/introducing-llamafile/))

Naszym celem jest uczynienie otwartych LLM znacznie bardziej dostępnymi zarówno dla deweloperów, jak i użytkowników końcowych. Osiągamy to poprzez połączenie [llama.cpp](https://github.com/ggerganov/llama.cpp) z [Cosmopolitan Libc](https://github.com/jart/cosmopolitan) w jedną ramę, która redukuje całą złożoność LLM do pojedynczego pliku wykonywalnego (nazwanego "llamafile"), który działa lokalnie na większości komputerów, bez instalacji.

![Mozilla](llamafile/mozilla-logo-bw-rgb.png)

llamafile jest projektem Mozilla Builders.


# llamafile.pl

llamafile.pl jest polską wersją projektu llamafile.


## Szybki start Bielika

1. Pobierz polską wersję [llamafile.pl](https://github.com/PolskiBielik/llamafile/releases/download/0.8.13-pl/llamafile.pl)

2. Otwórz terminal na swoim komputerze.

3. Jeśli używasz systemu macOS, Linux lub BSD, będziesz musiał nadać uprawnienia do wykonywania tego nowego pliku (robisz to tylko raz).

chmod +x llamafile.pl


4. Jeśli używasz systemu Windows, zmień nazwę pliku, dodając na końcu ".exe".

5. Uruchom plik llamafile.pl z wybranym plikiem modelu Bielika. Na przykład:

./llamafile.pl -m Bielik-11B-v2.3-Instruct.Q4_K_M.gguf --host 0.0.0.0

6. Twoja przeglądarka powinna automatycznie się otworzyć i wyświetlić interfejs czatu. (Jeśli tak się nie stanie, po prostu otwórz przeglądarkę i wpisz http://localhost:8080).

7. Gdy skończysz rozmawiać, wróć do terminala i naciśnij `Control-C`, aby zamknąć llamafile.pl.


## Problemy i rozwiązywanie problemów

Na dowolnej platformie, jeśli proces llamafile.pl jest natychmiast zabijany, sprawdź, czy masz zainstalowane CrowdStrike, a następnie poproś o dodanie do białej listy.


### Mac OS

Na macOS z procesorami Apple Silicon musisz mieć zainstalowane Narzędzia wiersza poleceń Xcode, aby llamafile.pl mógł się zainicjować.

Jeśli używasz zsh i masz problemy z uruchomieniem llamafile.pl, spróbuj wpisać `sh -c ./llamafile.pl`. 
Jest to spowodowane błędem, który został naprawiony w zsh 5.9+. Ta sama sytuacja dotyczy Pythona `subprocess`, 
starszych wersji Fish i innych.


#### Mac error "... cannot be opened because the developer cannot be verified"

    Natychmiast uruchom Ustawienia systemowe, a następnie przejdź do Prywatność i Bezpieczeństwo. llamafile.pl powinien być wyświetlony na dole, z przyciskiem do zezwolenia.
    Jeśli nie, zmień swoje polecenie w Terminalu na `sudo spctl --master-disable; [llama launch command]; sudo spctl --master-enable`. 
    Jest tak, ponieważ `--master-disable` wyłącza wszystkie sprawdzanie, więc musisz je ponownie włączyć po zamknięciu llama.


### Linux

Na niektórych systemach Linux możesz napotkać błędy związane z run-detectors lub WINE. Jest to spowodowane rejestracjami binfmt_misc. Możesz to naprawić, dodając dodatkową rejestrację dla formatu pliku APE, którego używa llamafile.pl:

```sh
sudo wget -O /usr/bin/ape https://cosmo.zip/pub/cosmos/bin/ape-$(uname -m).elf
sudo chmod +x /usr/bin/ape
sudo sh -c "echo ':APE:M::MZqFpD::/usr/bin/ape:' >/proc/sys/fs/binfmt_misc/register"
sudo sh -c "echo ':APE-jart:M::jartsr::/usr/bin/ape:' >/proc/sys/fs/binfmt_misc/register"
```

### Windows

Jak wspomniano powyżej, na Windows możesz potrzebować zmiany nazwy pliku llamafile.pl, dodając do niej .exe.

Jak wspomniano powyżej, Windows ma również limit rozmiaru pliku wykonywalnego wynoszący 4 GB. Wykonywalny plik serwera LLaVA powyżej jest tylko o 30 MB mniejszy od tego limitu, więc będzie działać na Windows, ale z większymi modelami, takimi jak WizardCoder 13B, musisz przechowywać wagi w osobnym pliku. Przykład jest podany powyżej; zobacz "Używanie llamafile.pl z zewnętrznymi wagami."

Na WSL może wystąpić wiele potencjalnych problemów. Jedną rzeczą, która może całkowicie rozwiązać te problemy, jest to:

```
[Unit]
Description=cosmopolitan APE binfmt service
After=wsl-binfmt.service

[Service]
Type=oneshot
ExecStart=/bin/sh -c "echo ':APE:M::MZqFpD::/usr/bin/ape:' >/proc/sys/fs/binfmt_misc/register"

[Install]
WantedBy=multi-user.target
```

Umieść to w  `/etc/systemd/system/cosmo-binfmt.service`.

Następnie uruchom `sudo systemctl enable cosmo-binfmt`.

Inną rzeczą, która pomogła użytkownikom WSL, którzy napotykają problemy, jest wyłączenie funkcji interop win32:

```sh
sudo sh -c "echo -1 > /proc/sys/fs/binfmt_misc/WSLInterop"
```

W przypadku otrzymania komunikatu o błędzie Permission Denied podczas wyłączania interop przez CLI, można go trwale wyłączyć, dodając następujące ustawienie w /etc/wsl.conf:

```sh
[interop]
enabled=false
```

## Wspierane systemy operacyjne

llamafile.pl wspiera następujące systemy operacyjne, które wymagają minimalnej instalacji stock:

    Linux 2.6.18+ (tj. każda dystrybucja od RHEL5 około 2007 roku)
    Darwin (macOS) 23.1.0+ [1] (GPU jest obsługiwane tylko na ARM64)
    Windows 10+ (AMD64 tylko)
    FreeBSD 13+
    NetBSD 9.2+ (AMD64 tylko)
    OpenBSD 7+ (AMD64 tylko)

Na Windows llamafile.pl działa jako natywny przenośny plik wykonywalny. Na systemach UNIX llamafile.pl ekstrahuje mały program ładujący o nazwie ape do $TMPDIR/.llamafile.pl lub ~/.ape-1.9, który jest używany do mapowania Twojego modelu w pamięci.

[1] Wersje jądra Darwin 15.6+ powinny być wspierane, ale obecnie nie mamy sposobu na przetestowanie tego.


## Wspierane procesory

llamafile.pl wspiera następujące procesory:

    AMD64 procesory muszą mieć AVX. W przeciwnym razie llamafile.pl wyświetli komunikat o błędzie i odmówi uruchomienia. Oznacza to, że jeśli masz procesor Intel, musi to być Intel Core lub nowszy (około 2006+), a jeśli masz procesor AMD, musi to być K8 lub nowszy (około 2003+). Wsparcie dla AVX512, AVX2, FMA, F16C i VNNI jest warunkowo włączane w czasie wykonywania, jeśli masz nowszy procesor. Na przykład Zen4 ma bardzo dobre AVX512, które może przyspieszyć llamafile.pl BF16.

    ARM64 procesory muszą mieć ARMv8a+. Oznacza to, że wszystko od Apple Silicon do 64-bitowych Raspberry Pi będzie działać, pod warunkiem, że Twoje wagi zmieszczą się w pamięci.

Warto zauważyć, że tłumaczenie zachowuje oryginalne formatowanie, w tym nagłówki, kod i wyjaśnienia. Jeśli masz jakieś pytania lub potrzebujesz dodatkowych informacji na temat jakiejkolwiek części tego tekstu, chętnie pomogę Ci je wyjaśnić.
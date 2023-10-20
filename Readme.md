# Laboratorium 3 - sprawozdanie

Zadanie rozpocząłem od wydania polecenia, które pozwoliło mi utworzyć przestrzeń nazw *lab3*:
```
kubectl create ns lab3
```

Następnie wydałem komendę, która pozwoliła mi utworzyć plik *sidecar-pod.yaml*, zawierający wstępną konfigurację poda o nazwie *sidecar-pod* w przestrzeni nazw *lab3*, opartego o obraz *busybox*. Dodałem również komendę pozwalającą na zapis w pętli, co 5 sekund, wyniku działania polecenia *date* do pliku */var/log/date.log*. Wykorzystałem ścieżkę do powłoki systemowej z parametrem "-c", aby móc użyć instrukcji *while* i jednocześnie podać pełne polecenie w jednej linii:
```
kubectl run sidecar-pod --image=busybox -n lab3 -o yaml --dry-run=client -- /bin/sh -c "while true; do date >> /var/log/date.log; sleep 5; done" > sidecar-pod.yaml
```

W dalszej części zmodyfikowałem zawartość pliku *sidecar-pod.yaml*. Zmieniłem nazwę utworzonego kontenera na *busybox*, zamontowałem dla niego katalog *workdir* pod ścieżkę */var/log* oraz dodałem kolejny kontener o nazwie *nginx*, oparty o obraz *nginx*. Wskazałem port kontenera (80) oraz zamontowałem dla niego katalog *workdir* pod ścieżkę */usr/share/nginx/html*. Ostatnim elementem konfiguracji, który dodałem, było wskazanie montowanego woluminu typu *hostPath* pod ścieżką */var/log*. Jako typ wybrałem *DirectoryOrCreate*, gdyż zgodnie z dokumentacją, ta opcja umożliwia utworzenie pustego katalogu w podanej ścieżce z odpowiednimi uprawnieniami, jeśli w podanej ścieżce nic nie istnieje.

Tak przygotowany plik posłużył mi do uruchomienia wielokontenerowego poda:
```
kubectl create -f sidecar-pod.yaml 
```

W dalszej części zweryfikowałem, czy pod rzeczywiście został utworzony, poleceniem:
```
kubectl get pods -n lab3 -o wide

NAME          READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
sidecar-pod   2/2     Running   0          35s   10.244.0.5   minikube   <none>           <none>
```

Sprawdziłem również zdarzenia, aby upewnić się, że nastąpiło pomyślne uruchomienie poda:
```
kubectl describe pod sidecar-pod -n lab3

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  28s   default-scheduler  Successfully assigned lab3/sidecar-pod to minikube
  Normal  Pulling    27s   kubelet            Pulling image "busybox"
  Normal  Pulled     25s   kubelet            Successfully pulled image "busybox" in 1.686947995s (1.686962751s including waiting)
  Normal  Created    25s   kubelet            Created container busybox
  Normal  Started    25s   kubelet            Started container busybox
  Normal  Pulling    25s   kubelet            Pulling image "nginx"
  Normal  Pulled     24s   kubelet            Successfully pulled image "nginx" in 1.723587104s (1.723596662s including waiting)
  Normal  Created    23s   kubelet            Created container nginx
  Normal  Started    23s   kubelet            Started container nginx
```

W dalszej części nawiązałem połączenie typu port-forwarding oraz przy użyciu programu *curl*, zweryfikowałem poprawność działania poda:
```
kubectl port-forward sidecar-pod -n lab3 8080:80 &
curl localhost:8080/date.log

Handling connection for 8080
Fri Oct 20 16:20:55 UTC 2023
Fri Oct 20 16:21:00 UTC 2023
Fri Oct 20 16:21:05 UTC 2023
Fri Oct 20 16:21:10 UTC 2023
Fri Oct 20 16:21:15 UTC 2023
Fri Oct 20 16:21:20 UTC 2023
Fri Oct 20 16:21:25 UTC 2023
```

Cel zadania został osiągnięty.

# Passo 1 - Crie o Container e Faça o Teste de Carga
```
docker run --rm --network=host jordi/ab -n 2000 -c 100 http://localhost:84/
```
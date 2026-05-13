# Lam moi anh cho media pod

## Muc tieu

Dam bao anh duoc hien thi dung trong ung dung bang cach copy lai du lieu mau vao pod `media`.

## Lenh thuc hien

```bash
MEDIA_POD=$(kubectl get pod -n yas | grep '^media-' | awk '{print $1}' | head -1)

kubectl exec -n yas "$MEDIA_POD" -- sh -c 'rm -rf /images && mkdir -p /images'

kubectl cp /workspace/yas/sampledata/images/sample yas/"$MEDIA_POD":/images/sample
```

## Ghi chu

- Can chay sau khi cluster da deploy xong va pod `media` dang running.

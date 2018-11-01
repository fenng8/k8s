一、CEPH介绍及配置
  1,Ceph基础知识和基础架构认识(https://www.cnblogs.com/luohaixian/p/8087591.html)
  2,Ceph(存储之块设备、文件系统、对象存储)(https://blog.csdn.net/fuyinggui/article/details/79425031)
  3,Ceph集群搭建(https://www.lijiawang.org/posts/intsall-ceph.html)

二、K8S配置
  1，Node安装ceph-common(需要epel)
  cat >/etc/yum.repos.d/ceph.repo <EOF
    [ceph]
    name=Ceph packages for $basearch
    baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/$basearch
    enabled=1
    gpgcheck=1
    priority=1
    type=rpm-md
    gpgkey=http://mirrors.163.com/ceph/keys/release.asc

    [ceph-noarch]
    name=Ceph noarch packages
    baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/noarch
    enabled=1
    gpgcheck=1
    priority=1
    type=rpm-md
    gpgkey=http://mirrors.163.com/ceph/keys/release.asc

    [ceph-source]
    name=Ceph source packages
    baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/SRPMS
    enabled=0
    gpgcheck=1
    type=rpm-md
    gpgkey=http://mirrors.163.com/ceph/keys/release.asc
    priority=1
  EOF
  yum install ceph-common -y

  2, 创建ceph-secret.yaml文件
  1) 登录ceph集群查看auth list
    ceph auth list
    查看client.admin的key: AQBMRURbehiZEBAAu0WHLzczjns8Zo2UYJaiNg==
    使用base64加密后: 
      echo -n "AQBMRURbehiZEBAAu0WHLzczjns8Zo2UYJaiNg==" | base64
      QVFCTVJVUmJlaGlaRUJBQXUwV0hMemN6am5zOFpvMlVZSmFpTmc9PQ==

    所以ceph-secret.yaml文件如下:
      apiVersion: v1
      kind: Secret
      metadata:
        name: ceph-secret-admin
      type: "kubernetes.io/rbd"
      data:
        key: QVFCTVJVUmJlaGlaRUJBQXUwV0hMemN6am5zOFpvMlVZSmFpTmc9PQ==

  2) 新建一个ceph-storageclass.yaml:
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: rbd
      provisioner: ceph.com/rbd
      parameters:
        monitors: 192.168.100.247:6789
        adminId: admin
        adminSecretName: ceph-secret-admin
        adminSecretNamespace: default
        pool: k8s
        userId: admin
        userSecretName: ceph-secret-admin
        fsType: xfs
        imageFormat: "2"
        imageFeatures: "layering"

  3) 创建一个rbd-provisioner
    1.11版本中的解决办法：
    k8s中使用kubeadm部署的v1.11.2版本中的kube-controller-manager pod中未安装rbd程序，导致无法使用rbd块存储。解决办法：https://github.com/kubernetes-incubator/external-storage/tree/master/ceph/rbd
git clone后kubectl create -f deploy/rbac image镜像为:quay.io/external_storage/rbd-provisioner:latest(v2.1.1-k8s1.11版本)目录后通过手动创建pvc以验证是否成功生效。



  

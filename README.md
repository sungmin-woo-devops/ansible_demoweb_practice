# Ansible Demoweb Practice

RedHat/CentOS 계열에서 데모 웹 서버(httpd)를 신속히 구성하는 Ansible Role입니다.
패키지 설치, 서비스 기동/부팅 연결, 방화벽 규칙(HTTP/HTTPS) 적용을 자동화합니다.

## Features
- OS 체크(예: CentOS가 아닌 경우 실패 처리)
- `httpd`, `mod_ssl`, `firewalld` 설치
- `httpd`, `firewalld`, 서비스 enable & start
- 방화벽 서비스 규칙 `http`, `https` 활성화
- 기본 인덱스 템플릿 배포: `template/index.html`

## Supported Platforms

## Role Variables
기본값(예: `vars/main.yml`
```yaml
pkg_web: httpd
pkg_web_https: mod_ssl
pkg_fw: firewalld
svc_web: httpd
svc_fw: firewalld
fwrule_http: http
fwrule_https: https
```

## Directory Layout
```bash
.
├─ tasks/
│  └─ main.yml              # 패키지 설치, 서비스 기동, 방화벽 오픈 등
├─ vars/
│  └─ main.yml              # 패키지/서비스/방화벽 변수
├─ templates/
│  └─ index.html            # 데모 페이지
├─ tests/
│  ├─ inventory
│  └─ test.yml              # 간단 테스트 플레이북(역할명 불일치 주의)
└─ meta/
   └─ main.yml              # galaxy_info
```

## Quick Start
1) 인벤토리
    ```ini
    [web]
    your-centos-host ansible_host=1.2.3.4
    ```
2) 플레이북
    ```yaml
    - name: Setup demo web
      hosts: web
      become: true
      roles:
        - role: demoweb    # 역할명 통일 필요(예: demoweb 또는 wsmweb)
    ```
3) 실행
    ```bash
    ansible-playbook -i tests/inventory tests/test.yaml
    # 또는
    ansible-playbook -i inventory playbook.yml
    ```

## Notes & Tips
- 역할명 통일: 저장소/디렉터리/테스트에서 wsmweb과 demoweb이 혼재. 하나로 통일하세요.
  예) 저장소명: `ansible-role-demoweb`, 역할 디렉터리: `roles/demoweb`

- 템플릿 배포 태스크: `template/index.html`을 실제로 배포하려면 `template` 또는 `copy` 태스크를
  `task/main.yml`에 추가하세요.

  ```yaml
  - name: Deploy demo index.html
    ansible.builtin.templates:
      src: index.html
      dest: /var/www/html/index.html
      owner: root
      group: root
      mode: '0644'
  ```

- 플랫폼 검사 보완: `ansible_facts['os_family']` 기반으로 RedHat 계열을 허용하도록 일반화하면 호환성이 좋아집니다.
  ```yaml
  - name: Fail if not RedHat Family
    ansible.builtin.fail:
      msg: "RedHat family only."
    when: ansible_facts['os_family'] ~= 'RedHat'
  ```

- 방화벽 미사용 환경 대응: firewalld 미사용 시 `iptables/nftables`로 분기하거나 태그로 제어하세요.

## License
GNU (원문 참조)

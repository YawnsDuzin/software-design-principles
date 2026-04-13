# Facade 패턴 (퍼사드)

> **핵심 의도 한줄 요약:** 복잡한 서브시스템에 대한 단순하고 통합된 인터페이스를 제공하여, 사용을 쉽게 만드는 패턴

---

## 목차

1. [개요](#1-개요)
2. [왜 필요한가? - 문제 상황](#2-왜-필요한가---문제-상황)
3. [비유로 이해하기](#3-비유로-이해하기)
4. [UML 다이어그램](#4-uml-다이어그램)
5. [패턴 적용 전 (Before)](#5-패턴-적용-전-before)
6. [패턴 적용 후 (After)](#6-패턴-적용-후-after)
7. [Python 구현](#7-python-구현)
8. [C# 구현](#8-c-구현)
9. [실무 예제: 주문 처리 퍼사드](#9-실무-예제-주문-처리-퍼사드)
10. [장단점](#10-장단점)
11. [관련 패턴](#11-관련-패턴)
12. [정리 및 체크리스트](#12-정리-및-체크리스트)

---

## 1. 개요

Facade 패턴은 **GoF 구조 패턴** 중 가장 직관적인 패턴입니다. 복잡한 서브시스템의 내부 동작을 숨기고, 클라이언트에게 **단순한 진입점(Entry Point)**을 제공합니다.

핵심 아이디어: "복잡한 것을 쉽게, 어려운 것을 간단하게"

실무에서 마주치는 상황:
- 여러 마이크로서비스를 하나의 API로 통합
- 복잡한 초기화 과정을 하나의 메서드로 단순화
- 여러 단계의 비즈니스 로직을 하나의 유즈케이스로 묶기

---

## 2. 왜 필요한가? - 문제 상황

### 상황: 영상 변환 시스템

영상 파일을 변환하려면 여러 라이브러리를 조합해야 합니다:

```
영상 변환을 위해 필요한 작업:
1. 비디오 코덱 설정
2. 오디오 코덱 설정
3. 비트레이트 조정
4. 해상도 변환
5. 포맷 변환기 초기화
6. 임시 파일 생성
7. 변환 실행
8. 임시 파일 정리
```

클라이언트가 이 모든 과정을 직접 관리해야 한다면?

```python
# 클라이언트가 8개의 서브시스템을 직접 알아야 함
video_codec = VideoCodec("h264")
audio_codec = AudioCodec("aac")
bitrate = BitrateManager(video_codec)
bitrate.set_rate(4500)
resolution = ResolutionConverter()
resolution.set_target(1920, 1080)
converter = FormatConverter(video_codec, audio_codec)
temp = TempFileManager()
temp_path = temp.create()
converter.convert(input_file, temp_path)
# ... 에러 처리는?
```

**문제:** 클라이언트가 서브시스템의 세부 사항을 모두 알아야 하고, 순서도 맞춰야 합니다.

---

## 3. 비유로 이해하기

### 컴퓨터의 전원 버튼

```
전원 버튼을 누르면 (Facade):

내부에서 일어나는 일 (서브시스템):
1. PSU(전원 공급 장치) 활성화
2. CPU 초기화
3. RAM 체크
4. BIOS 실행
5. 부트로더 로드
6. OS 커널 시작
7. 드라이버 로드
8. 서비스 시작
9. 로그인 화면 표시

사용자: 버튼 하나만 누르면 됨!
사용자가 PSU나 BIOS를 알 필요 없음!

┌─────────────────────────────────────┐
│        전원 버튼 (Facade)           │
│         [  ON  ]                    │
└────────────┬────────────────────────┘
             │
     ┌───────┴────────────────────┐
     │    복잡한 서브시스템들       │
     │  ┌─────┐ ┌─────┐ ┌─────┐  │
     │  │ PSU │ │ CPU │ │ RAM │  │
     │  └─────┘ └─────┘ └─────┘  │
     │  ┌──────┐ ┌──────┐        │
     │  │ BIOS │ │  OS  │        │
     │  └──────┘ └──────┘        │
     └────────────────────────────┘
```

### 호텔 프론트 데스크

```
고객: "체크인하고 싶습니다"

프론트 데스크 (Facade):
1. 예약 시스템에서 예약 확인
2. 결제 시스템에서 보증금 처리
3. 객실 관리 시스템에서 방 배정
4. 키카드 시스템에서 카드 발급
5. 미니바 시스템 초기화

고객은 프론트 데스크에만 말하면 됨!
내부 시스템을 하나도 알 필요 없음!
```

---

## 4. UML 다이어그램

```
┌──────────┐         ┌──────────────────────────────────────────┐
│  Client  │         │              Facade                      │
│          │────────>│──────────────────────────────────────────│
│          │         │ + simpleOperation()                      │
└──────────┘         │                                          │
                     │   1. subsystemA.operationA()             │
                     │   2. subsystemB.operationB()             │
                     │   3. subsystemC.operationC()             │
                     └────────┬─────────┬─────────┬─────────────┘
                              │         │         │
                     ┌────────┴──┐ ┌────┴────┐ ┌──┴──────────┐
                     │SubsystemA │ │SubsystemB│ │SubsystemC  │
                     │───────────│ │─────────│ │─────────────│
                     │+operationA│ │+operationB│ │+operationC│
                     └───────────┘ └─────────┘ └─────────────┘
```

**핵심:**
- Facade는 서브시스템들을 알고 있지만, 서브시스템들은 Facade를 모릅니다.
- 클라이언트는 Facade만 알면 됩니다.
- 서브시스템에 직접 접근하는 것도 **여전히 가능**합니다 (Facade는 강제가 아니라 제안).

---

## 5. 패턴 적용 전 (Before)

```python
# ❌ Before: 클라이언트가 서브시스템의 모든 세부사항을 직접 관리

class InventoryService:
    def check_stock(self, product_id: str) -> bool:
        print(f"[재고] {product_id} 재고 확인")
        return True

    def reserve_stock(self, product_id: str, qty: int):
        print(f"[재고] {product_id} x {qty} 예약")

    def release_stock(self, product_id: str, qty: int):
        print(f"[재고] {product_id} x {qty} 예약 해제")


class PaymentService:
    def validate_card(self, card_number: str) -> bool:
        print(f"[결제] 카드 유효성 검사")
        return True

    def charge(self, card_number: str, amount: int) -> str:
        print(f"[결제] {amount:,}원 결제")
        return "TXN-001"

    def refund(self, transaction_id: str):
        print(f"[결제] {transaction_id} 환불")


class ShippingService:
    def calculate_shipping(self, address: str) -> int:
        print(f"[배송] 배송비 계산")
        return 3000

    def create_shipment(self, address: str, product_id: str) -> str:
        print(f"[배송] 배송 생성")
        return "SHIP-001"


class NotificationService:
    def send_order_confirmation(self, email: str, order_id: str):
        print(f"[알림] {email}에 주문 확인 메일 발송")

    def send_shipping_notification(self, email: str, tracking: str):
        print(f"[알림] {email}에 배송 알림 발송")


# 클라이언트 코드: 서브시스템을 모두 직접 다룸
def place_order(product_id, qty, card_number, address, email):
    inventory = InventoryService()
    payment = PaymentService()
    shipping = ShippingService()
    notification = NotificationService()

    # 1. 재고 확인
    if not inventory.check_stock(product_id):
        raise Exception("재고 부족")

    # 2. 카드 검증
    if not payment.validate_card(card_number):
        raise Exception("카드 오류")

    # 3. 배송비 계산
    shipping_cost = shipping.calculate_shipping(address)

    # 4. 재고 예약
    inventory.reserve_stock(product_id, qty)

    # 5. 결제
    try:
        txn_id = payment.charge(card_number, 50000 + shipping_cost)
    except Exception:
        inventory.release_stock(product_id, qty)  # 롤백!
        raise

    # 6. 배송 생성
    try:
        tracking = shipping.create_shipment(address, product_id)
    except Exception:
        payment.refund(txn_id)  # 롤백!
        inventory.release_stock(product_id, qty)  # 롤백!
        raise

    # 7. 알림 발송
    notification.send_order_confirmation(email, "ORD-001")
    notification.send_shipping_notification(email, tracking)

    return "ORD-001"

# 문제: 다른 곳에서도 주문을 처리하면 이 로직을 복붙해야 함!
# 에러 처리와 롤백 로직도 반복!
```

---

## 6. 패턴 적용 후 (After)

```python
# ✅ After: Facade가 복잡한 흐름을 캡슐화

class OrderFacade:
    """주문 처리 퍼사드: 복잡한 서브시스템을 단순한 인터페이스로 제공"""

    def __init__(self):
        self._inventory = InventoryService()
        self._payment = PaymentService()
        self._shipping = ShippingService()
        self._notification = NotificationService()

    def place_order(self, product_id: str, qty: int,
                    card_number: str, address: str,
                    email: str) -> str:
        """주문 처리: 한 메서드로 모든 과정을 처리"""
        # 내부적으로 서브시스템을 올바른 순서로 호출
        # 에러 처리와 롤백도 여기서 관리
        ...
        return "ORD-001"


# 클라이언트: 단순!
facade = OrderFacade()
order_id = facade.place_order("PROD-1", 2, "4111...", "서울", "a@b.com")
```

---

## 7. Python 구현

```python
from dataclasses import dataclass
from enum import Enum
from typing import Optional


# ===== 서브시스템 1: 영상 코덱 =====
class VideoCodecType(Enum):
    H264 = "h264"
    H265 = "h265"
    VP9 = "vp9"


class VideoCodec:
    """영상 코덱 서브시스템"""

    def __init__(self, codec_type: VideoCodecType):
        self._type = codec_type
        print(f"[VideoCodec] {codec_type.value} 코덱 초기화")

    def configure(self, bitrate: int, keyframe_interval: int = 30):
        print(f"[VideoCodec] 비트레이트: {bitrate}kbps, "
              f"키프레임: {keyframe_interval}프레임")

    def encode(self, raw_data: bytes) -> bytes:
        print(f"[VideoCodec] {self._type.value}로 인코딩 중...")
        return b"encoded_video_data"


# ===== 서브시스템 2: 오디오 코덱 =====
class AudioCodec:
    """오디오 코덱 서브시스템"""

    def __init__(self, codec_type: str = "aac"):
        self._type = codec_type
        print(f"[AudioCodec] {codec_type} 코덱 초기화")

    def set_sample_rate(self, rate: int = 44100):
        print(f"[AudioCodec] 샘플레이트: {rate}Hz")

    def set_channels(self, channels: int = 2):
        print(f"[AudioCodec] 채널: {channels}")

    def encode(self, raw_audio: bytes) -> bytes:
        print(f"[AudioCodec] {self._type}로 인코딩 중...")
        return b"encoded_audio_data"


# ===== 서브시스템 3: 해상도 변환 =====
class ResolutionConverter:
    """해상도 변환 서브시스템"""

    PRESETS = {
        "480p": (854, 480),
        "720p": (1280, 720),
        "1080p": (1920, 1080),
        "4K": (3840, 2160),
    }

    def convert(self, source_width: int, source_height: int,
                target_width: int, target_height: int) -> dict:
        print(f"[Resolution] {source_width}x{source_height} -> "
              f"{target_width}x{target_height}")
        return {
            "width": target_width,
            "height": target_height,
            "method": "bicubic"
        }


# ===== 서브시스템 4: 파일 관리 =====
class FileManager:
    """임시 파일 관리 서브시스템"""

    def __init__(self):
        self._temp_files: list[str] = []

    def create_temp(self, extension: str) -> str:
        path = f"/tmp/convert_{id(self)}.{extension}"
        self._temp_files.append(path)
        print(f"[FileManager] 임시 파일 생성: {path}")
        return path

    def cleanup(self):
        for path in self._temp_files:
            print(f"[FileManager] 임시 파일 삭제: {path}")
        self._temp_files.clear()


# ===== 서브시스템 5: 포맷 합성기 =====
class FormatMuxer:
    """비디오/오디오 합성 서브시스템"""

    def mux(self, video_data: bytes, audio_data: bytes,
            output_format: str) -> bytes:
        print(f"[Muxer] 비디오 + 오디오 합성 -> {output_format}")
        return b"final_output_data"


# ===== Facade: 영상 변환기 =====
@dataclass
class ConvertOptions:
    """변환 옵션"""
    target_resolution: str = "1080p"
    video_codec: VideoCodecType = VideoCodecType.H264
    audio_codec: str = "aac"
    video_bitrate: int = 4500
    audio_sample_rate: int = 44100
    output_format: str = "mp4"


class VideoConverterFacade:
    """영상 변환 퍼사드

    복잡한 영상 변환 과정을 단순한 인터페이스로 제공합니다.

    내부적으로 5개의 서브시스템을 조합하여 사용합니다:
    - VideoCodec: 영상 인코딩
    - AudioCodec: 오디오 인코딩
    - ResolutionConverter: 해상도 변환
    - FileManager: 임시 파일 관리
    - FormatMuxer: 최종 합성
    """

    def convert(self, input_file: str,
                output_file: str,
                options: Optional[ConvertOptions] = None) -> bool:
        """영상 변환 실행

        Args:
            input_file: 입력 파일 경로
            output_file: 출력 파일 경로
            options: 변환 옵션 (미지정 시 기본값 사용)

        Returns:
            성공 여부
        """
        if options is None:
            options = ConvertOptions()

        file_mgr = FileManager()

        try:
            print(f"\n{'=' * 50}")
            print(f"영상 변환 시작: {input_file} -> {output_file}")
            print(f"{'=' * 50}")

            # 1. 코덱 초기화
            video = VideoCodec(options.video_codec)
            video.configure(options.video_bitrate)

            audio = AudioCodec(options.audio_codec)
            audio.set_sample_rate(options.audio_sample_rate)
            audio.set_channels(2)

            # 2. 해상도 변환
            resolution = ResolutionConverter()
            target = resolution.PRESETS.get(
                options.target_resolution, (1920, 1080)
            )
            resolution.convert(3840, 2160, *target)

            # 3. 임시 파일 생성
            temp_video = file_mgr.create_temp("raw_video")
            temp_audio = file_mgr.create_temp("raw_audio")

            # 4. 인코딩
            encoded_video = video.encode(b"raw_video_data")
            encoded_audio = audio.encode(b"raw_audio_data")

            # 5. 합성
            muxer = FormatMuxer()
            muxer.mux(encoded_video, encoded_audio,
                      options.output_format)

            print(f"\n변환 완료: {output_file}")
            return True

        except Exception as e:
            print(f"\n변환 실패: {e}")
            return False

        finally:
            # 6. 임시 파일 정리 (항상 실행)
            file_mgr.cleanup()

    def convert_quick(self, input_file: str,
                      output_file: str) -> bool:
        """빠른 변환 (720p, 기본 설정)"""
        return self.convert(
            input_file, output_file,
            ConvertOptions(target_resolution="720p",
                          video_bitrate=2500)
        )

    def convert_hd(self, input_file: str,
                   output_file: str) -> bool:
        """고화질 변환 (1080p, H.265)"""
        return self.convert(
            input_file, output_file,
            ConvertOptions(
                target_resolution="1080p",
                video_codec=VideoCodecType.H265,
                video_bitrate=6000
            )
        )


# ===== 사용 예시 =====
if __name__ == "__main__":
    # 클라이언트는 Facade만 알면 됨!
    converter = VideoConverterFacade()

    # 기본 변환
    converter.convert("input.avi", "output.mp4")

    # 빠른 변환
    converter.convert_quick("input.mov", "quick_output.mp4")

    # 고화질 변환
    converter.convert_hd("input.mkv", "hd_output.mp4")

    # 커스텀 옵션
    custom = ConvertOptions(
        target_resolution="4K",
        video_codec=VideoCodecType.VP9,
        output_format="webm"
    )
    converter.convert("input.mp4", "output.webm", custom)
```

---

## 8. C# 구현

```csharp
using System;
using System.Collections.Generic;

// ===== 서브시스템들 =====
public class InventoryService
{
    public bool CheckStock(string productId, int qty)
    {
        Console.WriteLine($"[재고] {productId} x {qty} 확인");
        return true;
    }

    public void ReserveStock(string productId, int qty)
    {
        Console.WriteLine($"[재고] {productId} x {qty} 예약");
    }

    public void ReleaseStock(string productId, int qty)
    {
        Console.WriteLine($"[재고] {productId} x {qty} 예약 해제");
    }
}

public class PaymentService
{
    public bool ValidatePaymentMethod(string cardNumber)
    {
        Console.WriteLine($"[결제] 카드 유효성 검사");
        return true;
    }

    public string ProcessPayment(string cardNumber, int amount)
    {
        Console.WriteLine($"[결제] {amount:N0}원 결제 처리");
        return $"TXN-{Guid.NewGuid().ToString()[..8]}";
    }

    public void Refund(string transactionId)
    {
        Console.WriteLine($"[결제] {transactionId} 환불 처리");
    }
}

public class ShippingService
{
    public int CalculateShippingCost(string address)
    {
        Console.WriteLine($"[배송] 배송비 계산: {address}");
        return 3000;
    }

    public string CreateShipment(string address, string productId,
                                  int qty)
    {
        Console.WriteLine($"[배송] 배송 생성: {productId} -> {address}");
        return $"SHIP-{Guid.NewGuid().ToString()[..8]}";
    }

    public string GetTrackingUrl(string shipmentId)
    {
        return $"https://tracking.example.com/{shipmentId}";
    }
}

public class NotificationService
{
    public void SendEmail(string to, string subject, string body)
    {
        Console.WriteLine($"[알림] 이메일 -> {to}: {subject}");
    }

    public void SendSms(string phone, string message)
    {
        Console.WriteLine($"[알림] SMS -> {phone}: {message}");
    }
}

public class DiscountService
{
    public int CalculateDiscount(string customerId, int amount)
    {
        Console.WriteLine($"[할인] 고객 등급별 할인 계산");
        return (int)(amount * 0.05);  // 5% 할인
    }
}

// ===== Facade =====
public class OrderResult
{
    public string OrderId { get; set; }
    public string TransactionId { get; set; }
    public string ShipmentId { get; set; }
    public int TotalAmount { get; set; }
    public int Discount { get; set; }
    public int ShippingCost { get; set; }
    public bool IsSuccess { get; set; }
    public string ErrorMessage { get; set; }
}

public class OrderFacade
{
    private readonly InventoryService _inventory;
    private readonly PaymentService _payment;
    private readonly ShippingService _shipping;
    private readonly NotificationService _notification;
    private readonly DiscountService _discount;

    public OrderFacade()
    {
        _inventory = new InventoryService();
        _payment = new PaymentService();
        _shipping = new ShippingService();
        _notification = new NotificationService();
        _discount = new DiscountService();
    }

    // DI 버전
    public OrderFacade(InventoryService inventory,
                       PaymentService payment,
                       ShippingService shipping,
                       NotificationService notification,
                       DiscountService discount)
    {
        _inventory = inventory;
        _payment = payment;
        _shipping = shipping;
        _notification = notification;
        _discount = discount;
    }

    /// <summary>
    /// 주문 처리: 복잡한 서브시스템 호출을 하나의 메서드로 캡슐화
    /// </summary>
    public OrderResult PlaceOrder(string customerId, string productId,
                                   int qty, int unitPrice,
                                   string cardNumber, string address,
                                   string email)
    {
        Console.WriteLine($"\n{"="u8.Repeat(50)}");
        Console.WriteLine($"주문 처리 시작: {productId} x {qty}");
        Console.WriteLine($"{"="u8.Repeat(50)}");

        string transactionId = null;
        string shipmentId = null;

        try
        {
            // 1. 재고 확인
            if (!_inventory.CheckStock(productId, qty))
            {
                return Fail("재고가 부족합니다.");
            }

            // 2. 결제 수단 검증
            if (!_payment.ValidatePaymentMethod(cardNumber))
            {
                return Fail("결제 수단이 유효하지 않습니다.");
            }

            // 3. 할인 계산
            int subtotal = unitPrice * qty;
            int discount = _discount.CalculateDiscount(
                customerId, subtotal);

            // 4. 배송비 계산
            int shippingCost = _shipping.CalculateShippingCost(address);

            // 5. 총 금액 계산
            int totalAmount = subtotal - discount + shippingCost;

            // 6. 재고 예약
            _inventory.ReserveStock(productId, qty);

            // 7. 결제 처리
            try
            {
                transactionId = _payment.ProcessPayment(
                    cardNumber, totalAmount);
            }
            catch (Exception)
            {
                _inventory.ReleaseStock(productId, qty);
                return Fail("결제 처리 중 오류가 발생했습니다.");
            }

            // 8. 배송 생성
            try
            {
                shipmentId = _shipping.CreateShipment(
                    address, productId, qty);
            }
            catch (Exception)
            {
                _payment.Refund(transactionId);
                _inventory.ReleaseStock(productId, qty);
                return Fail("배송 생성 중 오류가 발생했습니다.");
            }

            // 9. 알림 발송
            var orderId = $"ORD-{DateTime.Now:yyyyMMddHHmmss}";
            _notification.SendEmail(email,
                $"주문 확인: {orderId}",
                $"주문이 완료되었습니다. 금액: {totalAmount:N0}원");

            var trackingUrl = _shipping.GetTrackingUrl(shipmentId);
            _notification.SendSms("010-1234-5678",
                $"배송 추적: {trackingUrl}");

            Console.WriteLine($"\n주문 완료: {orderId}");

            return new OrderResult
            {
                OrderId = orderId,
                TransactionId = transactionId,
                ShipmentId = shipmentId,
                TotalAmount = totalAmount,
                Discount = discount,
                ShippingCost = shippingCost,
                IsSuccess = true
            };
        }
        catch (Exception ex)
        {
            // 예상치 못한 에러 시 롤백
            if (transactionId != null)
                _payment.Refund(transactionId);
            _inventory.ReleaseStock(productId, qty);
            return Fail($"예상치 못한 오류: {ex.Message}");
        }
    }

    private OrderResult Fail(string message)
    {
        return new OrderResult
        {
            IsSuccess = false,
            ErrorMessage = message
        };
    }
}

// ===== 사용 =====
public class Program
{
    public static void Main()
    {
        // Facade 하나로 복잡한 주문을 처리!
        var orderFacade = new OrderFacade();

        var result = orderFacade.PlaceOrder(
            customerId: "CUST-001",
            productId: "PROD-MacBook",
            qty: 1,
            unitPrice: 2500000,
            cardNumber: "4111-1111-1111-1111",
            address: "서울시 강남구",
            email: "user@example.com"
        );

        if (result.IsSuccess)
        {
            Console.WriteLine($"\n=== 주문 결과 ===");
            Console.WriteLine($"주문번호: {result.OrderId}");
            Console.WriteLine($"결제금액: {result.TotalAmount:N0}원");
            Console.WriteLine($"할인금액: {result.Discount:N0}원");
            Console.WriteLine($"배송비: {result.ShippingCost:N0}원");
        }
        else
        {
            Console.WriteLine($"\n주문 실패: {result.ErrorMessage}");
        }
    }
}
```

---

## 9. 실무 예제: 주문 처리 퍼사드

### Python - 이커머스 주문 퍼사드

```python
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
from typing import Optional
import uuid


class OrderStatus(Enum):
    CREATED = "created"
    PAID = "paid"
    SHIPPED = "shipped"
    COMPLETED = "completed"
    CANCELLED = "cancelled"
    FAILED = "failed"


@dataclass
class OrderItem:
    product_id: str
    product_name: str
    quantity: int
    unit_price: int


@dataclass
class OrderResult:
    success: bool
    order_id: Optional[str] = None
    total_amount: int = 0
    discount: int = 0
    shipping_cost: int = 0
    error: Optional[str] = None


# ===== 서브시스템들 =====
class ProductCatalog:
    """상품 카탈로그 서브시스템"""
    _products = {
        "P001": {"name": "맥북 프로", "price": 2500000, "stock": 10},
        "P002": {"name": "아이패드", "price": 1200000, "stock": 5},
        "P003": {"name": "에어팟", "price": 250000, "stock": 50},
    }

    def get_product(self, product_id: str) -> Optional[dict]:
        return self._products.get(product_id)

    def check_availability(self, product_id: str, qty: int) -> bool:
        product = self._products.get(product_id)
        if not product:
            return False
        return product["stock"] >= qty

    def reserve(self, product_id: str, qty: int):
        self._products[product_id]["stock"] -= qty
        print(f"  [카탈로그] {product_id} 재고 예약: {qty}개")

    def release(self, product_id: str, qty: int):
        self._products[product_id]["stock"] += qty
        print(f"  [카탈로그] {product_id} 재고 복원: {qty}개")


class PricingEngine:
    """가격 계산 서브시스템"""

    def calculate_subtotal(self, items: list[OrderItem]) -> int:
        return sum(item.unit_price * item.quantity for item in items)

    def apply_coupon(self, subtotal: int,
                     coupon_code: Optional[str]) -> int:
        discounts = {
            "WELCOME10": 0.10,
            "VIP20": 0.20,
            "SUMMER5": 0.05,
        }
        rate = discounts.get(coupon_code, 0)
        discount = int(subtotal * rate)
        if discount > 0:
            print(f"  [가격] 쿠폰 '{coupon_code}' 적용: "
                  f"-{discount:,}원 ({rate:.0%})")
        return discount

    def calculate_tax(self, amount: int) -> int:
        tax = int(amount * 0.1)
        print(f"  [가격] 부가세: {tax:,}원")
        return tax


class PaymentGateway:
    """결제 게이트웨이 서브시스템"""

    def authorize(self, card_number: str, amount: int) -> str:
        txn_id = f"TXN-{uuid.uuid4().hex[:8].upper()}"
        print(f"  [결제] 승인 완료: {txn_id}, {amount:,}원")
        return txn_id

    def capture(self, txn_id: str) -> bool:
        print(f"  [결제] 매입 완료: {txn_id}")
        return True

    def void(self, txn_id: str):
        print(f"  [결제] 승인 취소: {txn_id}")


class ShippingCalculator:
    """배송비 계산 서브시스템"""

    def calculate(self, address: str, total_weight: float) -> int:
        base_cost = 3000
        if "제주" in address or "도서" in address:
            base_cost += 3000
        weight_cost = int(total_weight * 100)
        total = base_cost + weight_cost
        print(f"  [배송] 배송비: {total:,}원 ({address})")
        return total

    def create_shipment(self, address: str,
                        items: list[OrderItem]) -> str:
        tracking = f"SHIP-{uuid.uuid4().hex[:8].upper()}"
        print(f"  [배송] 배송 접수: {tracking}")
        return tracking


class EmailNotifier:
    """이메일 알림 서브시스템"""

    def send_confirmation(self, email: str, order_id: str,
                          amount: int):
        print(f"  [이메일] 주문 확인 -> {email}")
        print(f"    주문번호: {order_id}, 금액: {amount:,}원")

    def send_receipt(self, email: str, txn_id: str):
        print(f"  [이메일] 영수증 -> {email} (거래: {txn_id})")


class FraudDetector:
    """사기 감지 서브시스템"""

    def check(self, card_number: str, amount: int,
              address: str) -> bool:
        # 실제로는 ML 모델 등을 사용
        is_safe = amount < 10_000_000
        status = "안전" if is_safe else "의심"
        print(f"  [사기감지] 결과: {status}")
        return is_safe


# ===== Facade: 주문 처리 =====
class OrderFacade:
    """주문 처리 퍼사드

    6개의 서브시스템을 조합하여 주문 프로세스를 단순화합니다.
    클라이언트는 이 클래스의 메서드만 호출하면 됩니다.
    """

    def __init__(self):
        self._catalog = ProductCatalog()
        self._pricing = PricingEngine()
        self._payment = PaymentGateway()
        self._shipping = ShippingCalculator()
        self._notifier = EmailNotifier()
        self._fraud = FraudDetector()

    def place_order(self, items: list[OrderItem],
                    card_number: str, address: str,
                    email: str,
                    coupon_code: Optional[str] = None) -> OrderResult:
        """주문 처리

        순서: 재고확인 -> 가격계산 -> 사기감지 -> 결제 -> 배송 -> 알림
        실패 시 자동 롤백
        """
        order_id = f"ORD-{datetime.now():%Y%m%d}-{uuid.uuid4().hex[:6].upper()}"
        txn_id = None
        reserved_items: list[OrderItem] = []

        try:
            print(f"\n{'=' * 60}")
            print(f"  주문 처리 시작: {order_id}")
            print(f"{'=' * 60}")

            # Step 1: 재고 확인 및 예약
            print("\n[Step 1] 재고 확인")
            for item in items:
                if not self._catalog.check_availability(
                        item.product_id, item.quantity):
                    return OrderResult(
                        success=False,
                        error=f"재고 부족: {item.product_name}"
                    )
                self._catalog.reserve(item.product_id, item.quantity)
                reserved_items.append(item)

            # Step 2: 가격 계산
            print("\n[Step 2] 가격 계산")
            subtotal = self._pricing.calculate_subtotal(items)
            discount = self._pricing.apply_coupon(subtotal, coupon_code)
            shipping_cost = self._shipping.calculate(address, 2.5)
            tax = self._pricing.calculate_tax(subtotal - discount)
            total = subtotal - discount + shipping_cost + tax

            print(f"  상품 합계: {subtotal:,}원")
            print(f"  할인: -{discount:,}원")
            print(f"  배송비: +{shipping_cost:,}원")
            print(f"  부가세: +{tax:,}원")
            print(f"  최종 금액: {total:,}원")

            # Step 3: 사기 감지
            print("\n[Step 3] 사기 감지")
            if not self._fraud.check(card_number, total, address):
                self._rollback_inventory(reserved_items)
                return OrderResult(
                    success=False,
                    error="결제가 보안 검사를 통과하지 못했습니다."
                )

            # Step 4: 결제
            print("\n[Step 4] 결제 처리")
            txn_id = self._payment.authorize(card_number, total)
            self._payment.capture(txn_id)

            # Step 5: 배송 접수
            print("\n[Step 5] 배송 접수")
            tracking = self._shipping.create_shipment(address, items)

            # Step 6: 알림 발송
            print("\n[Step 6] 알림 발송")
            self._notifier.send_confirmation(email, order_id, total)
            self._notifier.send_receipt(email, txn_id)

            print(f"\n{'=' * 60}")
            print(f"  주문 완료: {order_id}")
            print(f"{'=' * 60}")

            return OrderResult(
                success=True,
                order_id=order_id,
                total_amount=total,
                discount=discount,
                shipping_cost=shipping_cost
            )

        except Exception as e:
            print(f"\n[ERROR] 주문 실패: {e}")
            # 롤백
            if txn_id:
                self._payment.void(txn_id)
            self._rollback_inventory(reserved_items)
            return OrderResult(success=False, error=str(e))

    def _rollback_inventory(self, items: list[OrderItem]):
        """재고 롤백"""
        for item in items:
            self._catalog.release(item.product_id, item.quantity)


# ===== 사용 예시 =====
if __name__ == "__main__":
    facade = OrderFacade()

    # 주문 항목
    items = [
        OrderItem("P001", "맥북 프로", 1, 2500000),
        OrderItem("P003", "에어팟", 2, 250000),
    ]

    # 한 줄로 주문! (내부적으로 6개 서브시스템 조합)
    result = facade.place_order(
        items=items,
        card_number="4111-1111-1111-1111",
        address="서울시 강남구 테헤란로 123",
        email="customer@example.com",
        coupon_code="WELCOME10"
    )

    if result.success:
        print(f"\n주문 성공!")
        print(f"  주문번호: {result.order_id}")
        print(f"  총 금액: {result.total_amount:,}원")
        print(f"  할인: {result.discount:,}원")
    else:
        print(f"\n주문 실패: {result.error}")
```

---

## 10. 장단점

### 장점

| 장점 | 설명 |
|------|------|
| **사용 단순화** | 복잡한 서브시스템을 간단한 인터페이스로 사용 |
| **결합도 감소** | 클라이언트가 서브시스템에 직접 의존하지 않음 |
| **계층화 촉진** | 시스템을 계층적으로 구성하는 데 도움 |
| **유지보수 용이** | 서브시스템 변경이 Facade 내부에만 영향 |
| **일관된 에러 처리** | 에러 처리와 롤백을 한 곳에서 관리 |

### 단점

| 단점 | 설명 |
|------|------|
| **God Object 위험** | Facade가 너무 많은 기능을 가질 수 있음 |
| **유연성 제한** | 세밀한 제어가 필요한 경우 직접 접근이 필요 |
| **변경 집중** | 비즈니스 로직 변경 시 Facade를 자주 수정 |

### God Object 방지

```python
# ❌ 하나의 Facade에 모든 것을 넣지 마세요
class EverythingFacade:
    def place_order(self): ...
    def manage_inventory(self): ...
    def generate_report(self): ...
    def manage_users(self): ...  # 너무 많은 책임!

# ✅ 도메인별로 Facade를 분리하세요
class OrderFacade:
    def place_order(self): ...
    def cancel_order(self): ...

class InventoryFacade:
    def check_stock(self): ...
    def restock(self): ...

class ReportFacade:
    def generate_sales_report(self): ...
    def generate_inventory_report(self): ...
```

---

## 11. 관련 패턴

| 패턴 | 관계 |
|------|------|
| **Adapter** | 하나의 인터페이스를 변환 vs Facade는 여러 서브시스템을 단순화 |
| **Mediator** | 비슷한 역할이지만, Mediator는 양방향 통신, Facade는 단방향 |
| **Singleton** | Facade는 보통 하나만 필요하므로 Singleton으로 구현하기도 함 |
| **Abstract Factory** | Facade가 서브시스템 객체 생성을 Abstract Factory에 위임 가능 |

---

## 12. 정리 및 체크리스트

### 핵심 정리

```
Facade 패턴 = "복잡한 것을 단순하게"

When: 복잡한 서브시스템을 쉽게 사용해야 할 때
How:  서브시스템 호출을 하나의 클래스에 캡슐화
Why:  클라이언트 코드를 단순하게 유지하고, 결합도를 낮춤
```

### 적용 체크리스트

- [ ] 여러 서브시스템을 조합해야 하는가?
- [ ] 클라이언트가 서브시스템의 세부사항을 몰라도 되는가?
- [ ] 서브시스템에 대한 직접 접근도 필요한가? (Facade는 강제하지 않음)
- [ ] Facade가 너무 많은 책임을 갖고 있진 않은가? (God Object 주의)
- [ ] 에러 처리와 롤백을 Facade에서 관리하고 있는가?

### Facade vs Service Layer

| 구분 | Facade | Service Layer |
|------|--------|--------------|
| **범위** | 서브시스템 접근 단순화 | 비즈니스 로직 캡슐화 |
| **목적** | 복잡성 숨김 | 도메인 로직 제공 |
| **트랜잭션** | 선택적 | 보통 필수 |
| **계층** | 인프라 계층 | 응용 계층 |

> **기억하세요:** "Facade는 '있으면 편한 것'이지, '없으면 안 되는 것'이 아닙니다. 서브시스템에 직접 접근하는 것을 막지 않습니다."

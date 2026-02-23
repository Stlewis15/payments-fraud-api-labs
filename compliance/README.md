# 🛡️ Payment Compliance & Risk Mitigation

As a Solutions Engineer, my goal is to help merchants balance security with conversion. This section documents the technical strategies used to minimize a merchant's audit surface while maintaining a high-trust payment environment.

## 🏛️ PCI DSS v4.0: Scope Reduction Strategy
The transition from PCI v3.2.1 to v4.0 (mandatory as of March 2024) emphasizes a "Customized Approach." The most effective way to simplify this for a merchant is through **Scope Reduction**.

### SAQ-D vs. SAQ-A: The Business Impact
| Assessment Type | Typical Integration | Requirements | Effort |
| :--- | :--- | :--- | :--- |
| **SAQ-D** | Raw PAN handling / Direct API | 300+ | Very High (Annual Audit) |
| **SAQ-A-EP** | Partial Redirection / Elements | ~190 | Medium |
| **SAQ-A** | Hosted Iframes / Redirects | ~30 | Low (Self-Certify) |

### 🛠️ Technical Implementation: Hosted Fields
By implementing **Hosted Fields (Iframes)**, we ensure that sensitive card data (PAN) is captured by the payment provider’s script directly from the customer's browser.

**The SE Value-Add:**
* **Zero Data Touch:** The merchant server never sees the raw card data.
* **Tokenization:** The merchant receives a non-sensitive `token` to represent the transaction.
* **Script Security:** Meets new v4.0 requirements for monitoring and managing all JavaScript on the payment page.

---
*Reference: These frameworks were mastered during the Fraud & PCI-DSS Masterclass by Vasco Patrício.*

